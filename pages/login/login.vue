<template>
	<view class="content-login">
		<!-- <view class="uni-title uni-common-pl"></view> -->
		<view class="uni-list">
			<view class="uni-list-cell">
				<view class="uni-list-cell-left">
					选择平台：
				</view>
				<view class="uni-list-cell-db">
					<picker @change="changePlatform" :disabled="false" range-key="text" :value="platformIndex" :range="platformList">
						<view class="uni-input">{{platformName}}</view>
					</picker>
				</view>
			</view>
		</view>
		
		<view class="input-group">
		    <view class="input-row border">
		        <text class="title">账号：</text>
		        <!-- <m-input class="m-input" type="text" :disabled="false"  clearable focus v-model="userInfo.usernamePlatForm" placeholder="请输入账号"></m-input> -->
						<input-autocomplete
								class="unit-item__input"
								v-model="userInfo.usernamePlatForm"
								placeholder="请输入账号"
								highlightColor="#FF0000"
								:stringList="autocompleteStringList"
								v-on:selectItem="selectItemD"/>
		    </view>
		    <view class="input-row">
		        <text class="title">密码：</text>
		        <m-input type="password" displayable v-model="userInfo.passwordPlatForm" placeholder="请输入密码"></m-input>
		    </view>
		</view>

		<view class="btn-row">
		    <button type="primary" class="primary" @tap="handleLogin">登录</button>
		</view>

		<!-- <view style="margin-top:10px; color:#1989fa; text-align: center;">
      <a :href="utils.zhushouUrl">
        <text>点击下载登录助手</text>
      </a>
    </view> -->

		<!-- <view>
			<view class="content">
				<view class="sub-title">登录说明:</view>
				<view v-for="(item,index) in loginDescription.description" :key="index" class="item-wrap">{{ item }}</view>
			</view>
		</view> -->
	</view>
</template>

<script>
import save from '@/utils/save'
import loginDescription from './loginDescription.json'
import { getUtils, getRemoteOptions,loginFuzhu } from '@/api/game'
import { getValueByIndex, getIndexByValue, formatServerList, formatLastServerList } from '@/utils/index'
import {mapState,mapMutations} from 'vuex'
import mInput from '../../components/m-input.vue'

export default {
	components:{
		mInput
	},
	data() {
		return {
			loginDescription: loginDescription,
			platformIndex: 0,
			platformName: '',
			account: '',
			password: '',
			current: 0,
			remoteOptions: {},
			platformList: [], // 平台信息
			utils: '',
			autocompleteStringList: [],
			flag: {
			    showServer: false
			},
			userInfo: {
			  usernamePlatForm: '', // 平台的用户名
				passwordPlatForm: '', // 平台的密码
				userId: '',
			  platform: 1, // 这个platform用在像辅助添加用户的时候
			  server: '',
			  loginType: 1 // 官方平台：1，taptap：2
			},
			loginInfo: { // 登录过程中需要的数据
				userId: '',
			},
			serverInfo: { // 服务器列表
        client_ip: '',
        server_list: [],
        last_server_list: []
      }		
		}
	},
	onLoad() {
		this.handleGetRemoteOptions()
		this.handleGetUtils()
	},
	methods: {
		//响应选择事件，接收选中的数据
    selectItemD(data) {
				this.userInfo.passwordPlatForm = data.password
    },

		// 获取远程选项
		handleGetRemoteOptions() {
			getRemoteOptions()
			.then(res => {
				this.remoteOptions = res
				if (this.$global.saleChannel===6) {
					this.platformList = res.platformShendao
				} else {
					this.platformList = res.platform
				}
				this.loadLoginInfo()
			})
			.catch(err => {
				console.log(err)
			})
		},

		// 获取Utils
		handleGetUtils() {
      getUtils().then(res => {
        this.utils = res
      }).catch(err => {
        console.log(err)
      })
    },
		

		// 登录辅助
		handleLogin() {
			const params = {
				uname: this.userInfo.usernamePlatForm,
				upwd: this.userInfo.passwordPlatForm,
				login_type: this.userInfo.loginType,
				youxi_name: 'jdxz'
			}
			loginFuzhu(params).then(res => {
				if (res.code === 200) {
					this.saveAccountList(this.userInfo.usernamePlatForm, this.userInfo.passwordPlatForm)
					this.userInfo.userId = res.data.userid
					this.serverInfo.server_list = formatServerList(res.data.server_info.server_list)
					this.serverInfo.last_server_list = formatLastServerList(res.data.server_info.last_server_list)
					this.flag.showServer = true
					this.saveLoginInfo()
					this.toMain()
				} else if (res.code === 403) {
					uni.showToast({
						title: res.data.msg,
						duration: 2000,
						icon: 'none'
					})
				}
			}).catch(err => {
        console.log(err)
      })
		},

		/* url参数处理*/
    parseParams(data) {
      const paramsArr = []
      for (const i in data) {
        var key = encodeURIComponent(i)
        var value = data[i] ? encodeURIComponent(data[i]) : ''
        if (data[i]) {
          paramsArr.push(key + '=' + value)
        }
      }
      return paramsArr.join('&')
    },

		// 将登录成功的用户名密码添加到autocompleteStringList中
		saveAccountList(username,password) {
			const indexUsername = this.autocompleteStringList.findIndex((item) => {
				return item.text === username
			})
			if (indexUsername === -1) {
				if (this.autocompleteStringList.length >= 5) {
					this.autocompleteStringList.pop()
				}
				const userObj = {
					text: username,
					password: password
				}
				this.autocompleteStringList.unshift(userObj)
			} else {
				this.autocompleteStringList[indexUsername].password = password
			}
		},
		
		// 读取记住的登录信息
    loadLoginInfo() {
      const gameLoginInfo = save.getGameLoginInfo()
      if (gameLoginInfo) {
        this.userInfo.platform = gameLoginInfo.platform
        this.userInfo.server = gameLoginInfo.server
        this.userInfo.usernamePlatForm = gameLoginInfo.usernamePlatForm
        this.userInfo.passwordPlatForm = gameLoginInfo.passwordPlatForm
        this.userInfo.userId = gameLoginInfo.userId
				this.flag.showServer = gameLoginInfo.showServer
				this.platformName = gameLoginInfo.platformName
				this.serverInfo = gameLoginInfo.serverInfo || {}
				if (Array.isArray(gameLoginInfo.autocompleteStringList)) this.autocompleteStringList = gameLoginInfo.autocompleteStringList
				this.initSaveData()
      }
    },

    // 存储登录信息到LocalStorage
    saveLoginInfo() {
      const gameLoginInfo = {
        platform: this.userInfo.platform,
        server: '',
        usernamePlatForm: this.userInfo.usernamePlatForm,
        passwordPlatForm: this.userInfo.passwordPlatForm,
        loginType: this.userInfo.loginType,
        userId: this.userInfo.userId,
				showServer: this.flag.showServer,
				platformName: this.platformName,
				serverInfo: this.serverInfo,
				autocompleteStringList: this.autocompleteStringList
			}
      save.setGameLoginInfo(gameLoginInfo)
    },

		// 跳转到主页
		toMain() {
			uni.reLaunch({
        url: '../home/home',
      })
		},
		
		// 选择平台
		changePlatform: function(e) {
			if (e.target.value !== -1) {
				this.platformIndex = e.target.value
			} else {
				this.platformIndex = 0
			}
			this.platformName = this.platformList[this.platformIndex].text
			this.userInfo.loginType = getValueByIndex(this.platformList, this.platformIndex)
		},

		// 加载后将存储的数据显示出来
		initSaveData() {
			this.platformIndex = getIndexByValue(this.platformList, this.userInfo.loginType)
			if (this.platformIndex !== -1) {
				this.platformName = this.platformList[this.platformIndex].text
			}
		}
	}
}
</script>

<style lang="css" scoped>
.flex-row {
	display: flex;
	align-items: center;
	justify-content: space-between;
}
.left {
	width: 70%;
}
.server-wrap {
	margin-top: 20upx;
}
.btn-center {
	display: flex;
	align-items: center;
	justify-content: center;
	margin-top: 20upx;
}
.attr-flex {
	width: 100%;
	display: flex;
	flex-wrap: wrap;
	justify-content: space-between;
}
.attr-flex-item {
	flex: 1;
	width: 33.3%;
	min-width: 33.3%;
	max-width: 33.3%;
}
.radio-flex {
	/* width: 100%; */
	display: flex;
}
.radio-flex-item {
	flex: 1;
	width: 32%;
	min-width: 32%;
	max-width: 32%;
}
.content {
  padding: 10rpx 30rpx;
	user-select: text;
}
.sub-title {
	font-weight: 600;
}
.item-wrap {
  color: #969799;
  padding-bottom: 20rpx;
}
.unit-item__input {
	text-align: left;
	flex:1;
	padding: 0 10upx;
	min-height: 50upx;
	line-height: 50upx;
	z-index: 1;
}
</style>
