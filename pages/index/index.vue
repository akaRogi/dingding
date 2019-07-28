<template>
<view class="container">
  <view class="page-body">
    <view class="page-body-info">
      <scroll-view class="device-list" scroll-y>
        <view
			class="item"
			v-for="(item, index) in list"
			:key="index"
			@click="jlWifi(item)"
		>
          <view class="list">
            <text>{{ item.SSID }}</text>
            <span class="wifi-icon">
              <span class="wifi-1"></span>
              <span class="wifi-2" :class="{'off' : item.strength < 2}"></span>
              <span class="wifi-3" :class="{'off' : item.strength < 3}"></span>
              <span class="wifi-4" :class="{'off' : item.strength < 4}"></span>
              <span class="lock" v-if="item.secure"></span>
            </span>
          </view>
        </view>
      </scroll-view>
    </view>
    <view class="btn-area">
      <button type="primary" @click="startSearch">搜索Wi-Fi</button>
      <!-- <button @click="stopSearch">停止搜索</button> -->
    </view>
  </view>
</view>

</template>

<script>
	export default {
		data() {
			return {
				list: [],
				BSSID: '',
				tiem: ''
			}
		},
		onLoad() {
			wx.startWifi({
			  success (res) {
				  wx.getWifiList({
					  success (e) {
					  	console.log(e)
					  }
				  })
				console.log(res.errMsg)
			  }
			})
		},
		methods: {
			jlWifi (data) {
				this.BSSID = data.BSSID
				uni.showToast({
					title: '记录wifi：' + data.SSID,
					icon: 'none',
					duration: 4000
				})
				let This = this
				this.tiem = setInterval(() => {
					console.log('每隔一分钟检测一次')
					This.startSearch()
					let off = true
					This.list.forEach((el, index) => {
						if (el.BSSID === This.BSSID) {
							off = false
						}
					})
					if (off) {
						console.log('没有wifi')
						clearInterval(this.time)
						setInterval(() => {
							uni.vibrateLong({
								success: function () {
									console.log('success');
								}
							})
						}, 500)
					}
				},10000)
			},
			startSearch() {
				const getWifiList = () => {
					wx.getWifiList({
						success: (res) => {
							console.log(res)
							wx.onGetWifiList((res) => {
								console.log(res)
								const wifiList = res.wifiList
								.sort((a, b) => b.signalStrength - a.signalStrength)
								.map(wifi => {
									const strength = Math.ceil(wifi.signalStrength * 4)
									return Object.assign(wifi, { strength })
								})
								this.list = res.wifiList
							})
						},
						fail(err) {
							console.error(err);
						}
				  })
				}

				const startWifi = () => {
					wx.startWifi({
						success: getWifiList,
						fail(err) {
							console.error(err);
						}
					})
				}

				wx.getSystemInfo({
					success(res) {
						const isIOS = res.platform === 'ios'
						if (isIOS) {
							wx.showModal({
								title: '提示',
								content: '由于系统限制，iOS用户请手动进入系统WiFi页面，然后返回小程序。',
								showCancel: false,
								success() {
									startWifi()
								}
							})
							return
						}
						startWifi()
					}
				})
			},
			stopSearch() {
				wx.stopWifi({
					success(res) {
						console.log(res);
					},
					fail(err) {
						console.error(err);
					}
				})
			}
		}
	}
</script>

<style>
.page-body-info {
  padding: 30upx 60upx;
  width: auto;
}

.page-body-info {
  display: flex;
  flex-direction: column;
  align-items: center;
  background-color: #fff;
  width: 100%;
  padding: 50upx 0 150upx 0;
}

.device-list {
  height: 300upx;
  text-align: center;
}

.item {
  width: 100%;
  margin-bottom: 10px;
}

.list {
  width: 100%;
  text-align: left;
  display:flex;
  flex-direction:row;
  align-items:center;
  justify-content:space-between;
  font-size: 30upx;
}

.list text {
  display: inline-block;
  max-width: 80%;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.strength-ready { color: #26a69a; }
.strength-no { color: #37474f; }
.wifi-icon{
  width: 20px;
  height:20px;
  display: inline-block;
  position: relative;
  overflow: hidden;
  margin: 0 10px;
  float: right;
}
.wifi-icon span{
  display: block;
  position: absolute;
  border-radius: 50%;
}
.wifi-icon .wifi-1{
  width: 4px;
  height: 4px;
  left: 8px;
  bottom: 3px;
  background: currentcolor;
}
.wifi-icon .wifi-2,.wifi-icon .wifi-3,.wifi-icon .wifi-4{
  border: 2px solid;
  border-color: currentcolor transparent transparent;
 }
.wifi-icon .wifi-2{
  width: 12px;
  height: 12px;
  left: 2px;
  bottom: -4px;
}
.wifi-icon .wifi-3{
  width: 20px;
  height: 20px;
  left: -2px;
  bottom: -8px;
  }
.wifi-icon .wifi-4{
  width: 28px;
  height: 28px;
  left: -6px;
  bottom: -12px;
}
.wifi-icon span:after{
  content: "";
  display: block;
  position: absolute;
}
.wifi-icon .lock{
  width: 6px;
  height: 5px;
  background: currentcolor;
  display: block;
  right: 1px;
  bottom: 2px;
  border-radius: 0;
}
.wifi-icon .lock:after{
  width: 4px;
  height: 3px;
  border: 1px solid;
  border-radius: 100px 100px 0 0;
  border-bottom: none;
  left: 0px;
  bottom: 100%;
}
.off {
  border-color: #b0bec5 transparent transparent !important;
}

</style>
