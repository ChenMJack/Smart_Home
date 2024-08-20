<template>
	<view class="wrap">
		<view class="dev-area">
			<view class="dev-card">
				<view class="">
					<view class="dev-name">温度</view>
					<image class="dev-logo" src="../../static/temp.png" mode=""></image>
				</view>
				<view class="">
					<view class="dev-data">{{temp}} °C</view>
				</view>
			</view>

			<view class="dev-card">
				<view class="">
					<view class="dev-name">湿度</view>
					<image class="dev-logo" src="../../static/humi.png" mode=""></image>
				</view>
				<view class="">
					<view class="dev-data">{{humi}} °C</view>
				</view>
			</view>

			<view class="dev-card">
				<view class="">
					<view class="dev-name">台灯</view>
					<image class="dev-logo" src="../../static/led.png" mode=""></image>
				</view>
				<switch :checked="led" @change="onLedSwitch" color="#2b9939"/>
			</view>
			
			<view class="dev-card">
				Take your time!
			</view>
		</view>
	</view>

</template>

<script>
	const {
		createCommonToken
	} = require('@/key.js')
	export default {
		data() {
			return {
				temp: '',
				humi: '',
				led: true,
				token: '',
			}
		},
		onLoad() {
			const params = {
				author_key: 'FxTBrhHlDbXELMDt2UdZ7TUmJxAgqrsnrnEv+uhBms3X/5gKYFaxhPHOtlrpNPuubjbK3yldTHEHK+J3pM6Ptw==',
				version: '2022-05-01',
				user_id: '401608',
			}

			this.token = createCommonToken(params);

		},
		onShow() {
			this.fetchDevData();
			setInterval(()=>{
				this.fetchDevData();
			},3000)
		},
		methods: {
			fetchDevData() {
				uni.request({
					url: 'https://iot-api.heclouds.com/thingmodel/query-device-property',
					method: 'GET',
					data: {
						product_id: 'lr515XX6ao',
						device_name: 'mqtt1'
					},
					header: {
						'authorization': this.token
					},
					success: (res) => {
						console.log(res.data);
						this.humi = res.data.data[0].value
						this.led = res.data.data[1].value === 'true';
						this.temp = res.data.data[2].value
					}
				});
			},
			onLedSwitch(event) {
				console.log(event.detail.value);
				let value = event.detail.value;
				uni.request({
					url: 'https://iot-api.heclouds.com/thingmodel/set-device-property',
					method: 'POST',
					data: {
						product_id: 'lr515XX6ao',
						device_name: 'mqtt1',
						params: {
							"led": value
						}
					},
					header: {
						'authorization': this.token
					},
					success: () => {
						console.log('LED ' + (value ? 'ON' : 'OFF') + ' !');
					}
				});
			}
		}
	}
</script>

<style>
	.wrap {
		padding: 30rpx;

	}

	.dev-area {
		display: flex;
		justify-content: space-between;
		flex-wrap: wrap;
	}

	.dev-card {
		height: 150rpx;
		width: 320rpx;
		border-radius: 30rpx;
		margin-top: 30rpx;
		display: flex;
		justify-content: space-around;
		align-items: center;
		box-shadow: 0 0 15rpx #ccc;
	}

	.dev-name {
		font-size: 20rpx;
		text-align: center;
		color: #6d6d6d;

	}

	.dev-logo {
		height: 70rpx;
		width: 70rpx;
		margin-top: 12rpx;
	}

	.dev-data {
		font-size: 50rpx;
		color: #6d6d6d;
	}
</style>