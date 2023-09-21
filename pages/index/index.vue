<template>
	<view>
		<view :class="weather_cls">
			<view class="weather_left">
				<view class="weather_leftl">
					<text>{{temperature}}°</text>
				</view>
				<view class="weather_leftr">
					<text>{{weather}}</text>
					<p>{{location}}</p>
				</view>
			</view>
			<view class="weather_right"></view>
		</view>
		<view class="wind">
			<view class="wind_card">
				<view class="wind_cardl">
					<text>风向：</text>
					<br />
					<br />
					<text>风力：</text>
				</view>
				<view class="wind_cardr">
					<text>{{winddirection}}</text>
					<br />
					<br />
					<text>{{windpower}}</text>
				</view>
			</view>
		</view>
		<view class="airquality">
			<view class="airquality_card">
				<view class="airquality_cardl">
					<text>aqi：</text>
					<br />
					<text>pm2.5：</text>
					<br />
					<text>pm10：</text>
					<br />
					<text>so2：</text>
					<br />
					<text>no2：</text>
					<br />
					<text>co：</text>
					<br />
					<text>o3：</text>
					<br />
					<text>空气质量：</text>
				</view>
				<view class="airquality_cardr">
					<text>{{aqi}}</text>
					<br />
					<text>{{pm25}}</text>
					<br />
					<text>{{pm10}}</text>
					<br />
					<text>{{so2}}</text>
					<br />
					<text>{{no2}}</text>
					<br />
					<text>{{co}}</text>
					<br />
					<text>{{o3}}</text>
					<br />
					<text>{{quality}}</text>
				</view>
			</view>
		</view>
		<echarts-el class="charts" :option="option1"></echarts-el>
		<echarts-el class="charts" :option="option2"></echarts-el>
	</view>
</template>

<script>
	import qs from 'querystring'
	import axios from '@/js_sdk/gangdiedao-uni-axios'
	
	import Echarts from '@/components/echarts/echarts.vue'
	import EchartsEl from '@/components/echarts/echarts-el.vue'

	var reqData = {
		lat: '',
		lon: '',
		token: 'b87cea6abc73f77148534e03adff4d09'
	}

	var xinzhiKey = '你的心知天气key'
	var gaodeKey = '你的高德地图key'
	export default {
		data() {
			return {
				location: '',
				weather: '',
				temperature: '',
				winddirection: '',
				windpower: '',
				aqi: '',
				pm25: '',
				pm10: '',
				so2: '',
				no2: '',
				co: '',
				o3: '',
				quality: '',
				weather_cls: 'weather weather-img-sunny',
				option1: {
					title: {
						text: '最近四天气温变化',
					},
					tooltip: {
						trigger: 'axis'
					},
					legend: {
						data: ['最高气温', '最低气温'],
						right:0
					},
					xAxis: {
						type: 'category',
						boundaryGap: false,
						data: ['今天', '明天', '后天', '大后天']
					},
					yAxis: {
						type: 'value',
						axisLabel: {
							formatter: '{value} °C'
						}
					},
					series: [{
							name: '最高气温',
							type: 'line',
							data: [],
							markPoint: {
								data: [{
										type: 'max',
										name: '最大值'
									},
									{
										type: 'min',
										name: '最小值'
									}
								]
							},
							markLine: {
								data: [{
									type: 'average',
									name: '平均值'
								}]
							}
						},
						{
							name: '最低气温',
							type: 'line',
							data: [],
							markPoint:{
								data: [{
										type: 'max',
										name: '最大值'
									},
									{
										type: 'min',
										name: '最小值'
									}
								]
							},
							markLine: {
								data: [{
									type: 'average',
									name: '平均值'
								}]
							}
						}
					]
				},
				option2: {
					title: {
						text: '最近四天湿度变化',
					},
					tooltip: {
						trigger: 'axis'
					},
					legend: {
						data: ['湿度'],
						right:5
					},
					xAxis: {
						type: 'category',
						boundaryGap: false,
						data: ['今天', '明天', '后天', '大后天']
					},
					yAxis: {
						type: 'value',
						axisLabel: {
							formatter: '{value} %'
						}
					},
					series: [{
							name: '湿度',
							type: 'line',
							data: [],
							markPoint: {
								data: [{
										type: 'max',
										name: '最大值'
									},
									{
										type: 'min',
										name: '最小值'
									}
								]
							},
							markLine: {
								data: [{
									type: 'average',
									name: '平均值'
								}]
							}
						}
					]
				}
			}
		},
		components: {
		 Echarts,
		 EchartsEl
		 },
		onLoad() {
			let that = this
			//页面加载就获取相关信息
			this.getData(that)
		},
		methods: {
			getData: async (that) => {
				//请求定位
				let res = await axios.get('https://restapi.amap.com/v3/ip?key=' + gaodeKey)
				const {
					city,
					adcode
				} = res.data
				that.location = city

				//请求空气质量指数
				axios.get(' https://api.seniverse.com/v3/air/now.json?key=' + xinzhiKey + '&location=' + city).then((
					res) => {
					const {
						aqi,
						pm25,
						pm10,
						so2,
						no2,
						co,
						o3,
						quality
					} = res.data.results[0].air.city
					that.aqi = aqi
					that.pm25 = pm25
					that.pm10 = pm10
					that.so2 = so2
					that.no2 = no2
					that.co = co
					that.o3 = o3
					that.quality = quality
				}).catch((err) => {
					uni.showToast({
						title: "获取失败！请点击下方按钮重新获取！",
						icon: 'none'
					})
				})

				//请求天气情况
				axios.get(
					'https://restapi.amap.com/v3/weather/weatherInfo?key=0fdba6429d9feb9b63240ac17aab5dc7&extensions=base&city=' +
					adcode).then((res) => {
					const {
						humidity,
						temperature,
						weather,
						winddirection,
						windpower
					} = res.data.lives[0]
					that.weather = weather
					that.temperature = temperature
					that.winddirection = winddirection,
						that.windpower = windpower
					if (weather.charAt(weather.length - 1) === '风') {
						that.weather_cls = 'weather weather-img-cloudy'
					} else if (weather.charAt(weather.length - 1) === '雨') {
						that.weather_cls = 'weather weather-img-rainy'
					} else {
						that.weather_cls = 'weather weather-img-sunny'
					}
				}).catch((error) => {
					uni.showToast({
						title: "获取失败！请点击下方按钮重新获取！",
						icon: 'none'
					})
				})

				//请求温度和适度
				axios.get(
					'https://api.seniverse.com/v3/weather/daily.json?key='+xinzhiKey+'&location='+city+'&language=zh-Hans&unit=c&start=0&days=4').then((res) => {
						const {daily} = res.data.results[0]
						let high = []
						let low = []
						let humidity = []
						daily.map((item)=>{
							high.push(item.high)
							low.push(item.low)
							humidity.push(item.humidity)
						})
						that.option1.series[0].data = high
						that.option1.series[1].data = low
						that.option2.series[0].data = humidity
					}).catch((err)=>{
						uni.showToast({
							title: "获取失败！请点击下方按钮重新获取！",
							icon: 'none'
						})
					})
			},
		}
	}
</script>


<style>
	.weather {
		width: 95%;
		margin: 0 auto;
		height: 300rpx;
		margin: 10px;

		background-size: cover;
		-webkit-background-size: cover;
		-moz-background-size: cover;
		-o-background-size: cover;
		-ms-background-size: cover;
		border-radius: 20px;
		-webkit-border-radius: 20px;
		-moz-border-radius: 20px;
		-o-border-radius: 20px;
		-ms-border-radius: 20px;
	}

	.weather-img-sunny {
		background: url(../images/1.jpg) no-repeat 0px 0px;
	}

	.weather-img-cloudy {
		background: url(../images/2.jpg) no-repeat 0px 0px;
	}

	.weather-img-rainy {
		background: url(../images/4.jpg) no-repeat 0px 0px;
	}

	.weather-img-Thunders {
		background: url(../images/3.jpg) no-repeat 0px 0px;
	}

	.wind {
		width: 95%;
		margin: 0 auto;
		height: 500rpx;
		margin: 10px;
		background: url(../images/windmill.png) no-repeat 0px 0px;
		background-size: cover;
		-webkit-background-size: cover;
		-moz-background-size: cover;
		-o-background-size: cover;
		-ms-background-size: cover;
		border-radius: 20px;
		-webkit-border-radius: 20px;
		-moz-border-radius: 20px;
		-o-border-radius: 20px;
		-ms-border-radius: 20px;
		position: relative
	}

	.airquality {
		width: 95%;
		margin: 0 auto;
		height: 600rpx;
		margin: 10px;
		background: url(../images/air.png) no-repeat 0px 0px;
		background-size: cover;
		-webkit-background-size: cover;
		-moz-background-size: cover;
		-o-background-size: cover;
		-ms-background-size: cover;
		border-radius: 20px;
		-webkit-border-radius: 20px;
		-moz-border-radius: 20px;
		-o-border-radius: 20px;
		-ms-border-radius: 20px;
		position: relative
	}

	.weather_left {
		float: left;
		padding: 2rem;
	}

	.weather_leftl {
		float: left;
	}

	.weather_leftl text {
		font-size: 3rem;
		font-weight: 900;
		color: #212121;
	}


	.weather_leftr {
		float: left;
		margin: .5rem 0 0 1rem;
	}

	.weather_leftr text {
		font-size: 1.2rem;
		color: #0070b9;
		font-weight: 600;
	}

	.weather_leftr p {
		font-size: 50rpx;
		color: #000;
	}

	.wind_card {
		position: absolute;
		margin: 150rpx 10rpx 20rpx 58%;
		display: flex;
	}

	.wind_cardl {
		font-size: 1.2rem;
		color: #555555;
		font-weight: 600;
	}

	.wind_cardr {
		font-size: 1.2rem;
		color: #000000;
		font-weight: 600;
		margin-left: 75rpx;
	}

	.airquality_card {
		position: absolute;
		margin: 60rpx 10rpx 50rpx 55%;
		display: flex;
	}

	.airquality_cardl {
		font-size: 1.2rem;
		color: #555555;
		font-weight: 600;
	}

	.airquality_cardr {
		font-size: 1.2rem;
		color: #000000;
		font-weight: 600;
	}
	.charts{
		margin: 10rpx 10rpx 10rpx 10rpx;
		height: 700rpx;
	}
</style>
