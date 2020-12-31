<template>
	<view class="content">		
		<view v-if="flag.showServer">
		  <view class="list-cell">
		      <view class="uni-list-cell-left">
		          有角色的服务器：
		      </view>
		      <view class="uni-list-cell-db">
		          <picker @change="changeLastServer" :value="lastServerIndex" class="bg-picker-gongfa" range-key="text" :range="serverInfo.last_server_list">
		              <view class="uni-input">{{serverName}}</view>
		          </picker>
		      </view>
		  </view>
			<view class="list-cell">
		      <view class="uni-list-cell-left">
		          收藏的角色：
		      </view>
		      <view class="uni-list-cell-db">
		          <picker @change="changeSavedRoleList" :value="roleIndex" class="bg-picker-gongfa" range-key="roleName" :range="roleList">
		              <view class="uni-input">{{selectRoleName}}</view>
		          </picker>
		      </view>
		  </view>
			<!-- <view class="uni-list-cell">
			    <view class="uni-list-cell-left">
			        所有的服务器：
			    </view>
			    <view class="uni-list-cell-db">
			        <picker @change="changeAllServer" :value="allServerindex" range-key="text" :range="serverInfo.server_list">
			            <view class="uni-input">{{serverInfo.server_list[allServerindex].text}}</view>
			        </picker>
			    </view>
			</view> -->
		</view>
		<view v-if="!flag.showServer" class="btn-row">
		    <button type="primary" @tap="handleLogin">登录</button>
		</view>
		<view v-else>
			<view class="btn-center btn-center-margin">
				<view>
					<button type="primary" plain="true" size="mini" @tap="transferTime">转移辅助</button>
					<text style="width: 10upx; display: inline-block;"></text>
					<button type="primary" plain="true" size="mini" @tap="loginSwitch">切换账号</button>
					<text style="width: 10upx; display: inline-block;"></text>
					<button type="primary" plain="true" size="mini" @tap="handleGetServerList">更新服务器</button>
				</view>
			</view>
			<view  class="btn-center">
				<view>
					<button type="primary" plain="true" size="mini" @tap="handleSaveRole">收藏角色</button>
					<text style="width: 10upx; display: inline-block;"></text>
					<button type="primary" plain="true" size="mini" @tap="handleRemoveRole">移除收藏</button>
					<text style="width: 10upx; display: inline-block;"></text>
					<button type="primary" plain="true" size="mini" @tap="handleClearSaveRole">清空收藏</button>
				</view>
			</view>
		</view>

		<text v-if="utils.showCommon" class="waring-wrap">{{ utils.common }}</text>
		<text v-if="utils.showContact&&saleChannel === 'test'" class="waring-wrap">{{ utils.contact }}</text>
		<text v-if="utils.showContact1&&saleChannel === '1'" class="waring-wrap">{{ utils.contact1 }}</text>
		<text v-if="utils.showContact2&&saleChannel === '2'" class="waring-wrap">{{ utils.contact2 }}</text>
		<text v-if="utils.showContact3&&saleChannel === '3'" class="waring-wrap">{{ utils.contact3 }}</text>
		<text v-if="utils.showContact4&&saleChannel === '4'" class="waring-wrap">{{ utils.contact4 }}</text>
		<text v-if="utils.showContact5&&saleChannel === '5'" class="waring-wrap">{{ utils.contact5 }}</text>
		<text v-if="utils.showContact6&&saleChannel === '6'" class="waring-wrap">{{ utils.contact6 }}</text>
		<text v-if="utils.showContact7&&saleChannel === '7'" class="waring-wrap">{{ utils.contact7 }}</text>
		<text v-if="utils.showContact8&&saleChannel === '8'" class="waring-wrap">{{ utils.contact8 }}</text>
		
		
		<view class="uni-divider">
			<view class="uni-divider__content">云挂机</view>
			<view class="uni-divider__line"></view>
		</view>
		
		<view>
			<text>平台：</text>
			<text>{{ this.platformName }}</text>
		</view>
		<view>
			<text>角色名：</text>
			<text>{{ roleInfo.role_name }}</text>
		</view>
		<view>
			<text>辅助版本类型：</text>
			<text>{{ roleInfo.fuzhu_vip | vipStatus }}</text>
		</view>
		<view>
			<text>辅助到期时间：</text>
			<text :class="{danger: fuzuStatus.isExpired}">{{ fuzuStatus.end_time }}</text>
		</view>
		<view>
			<text>数据更新时间：</text>
			<text :class="{danger: isPassedTwoHours}">{{ roleInfo.update_time }}</text>
		</view>
		<view class="fuzhu-info">
			<text>续费请提供此ID：</text>
			<text class="content-wrap">{{ userInfo.userId }}</text>
			<button class="copy-button" type="primary" plain="true" size="mini" @tap="doCopy">复制</button>
		</view>
		<view>
			<text>云挂机状态：</text>
			<text>{{ roleInfo.on_off | statusFilter }}</text>
		</view>
		
		<view class="btn-center">
			<view>
				<button v-if="roleInfo.on_off" type="warn" size="mini" @tap="handleStopguaji">停止云挂机</button>
				<button v-else type="primary" size="mini" @tap="handleStartguaji">开启云挂机</button>
				<text style="width: 10upx; display: inline-block;"></text>
				<button :loading="statusLoading" :disabled="statusLoading" type="primary" size="mini" @tap="handleGuajiStatus">获取最新数据</button>
			</view>
		</view>

		<view v-for="(displayInfo, displayIndex) in viewConfig.gameInfoConfig" :key="displayIndex">
			<view class="uni-divider">
				<view class="uni-divider__content">{{displayInfo.label}}</view>
				<view class="uni-divider__line"></view>
			</view>
			<view class="attr-flex">
				<view v-for="(item) in viewConfig[displayInfo.key]" :key="item.key" :class="getAttrClass(item.columnSize)">
					<text>{{ item.label }}</text>
					<text>{{ roleInfo[item.key] | valueFormatFilter}}</text>
				</view>
			</view>
		</view>
		
		<!-- <view class="uni-divider">
			<view class="uni-divider__content">角色信息</view>
			<view class="uni-divider__line"></view>
		</view>
		
		<view class="uni-divider">
			<view class="uni-divider__content">活动次数</view>
			<view class="uni-divider__line"></view>
		</view>
		
		<view class="attr-flex">
			<view v-for="(item) in viewConfig.taskInfoConfig" :key="item.key" :class="getAttrClass(item.columnSize)">
				<text>{{ item.label }}</text>
				<text>{{ roleInfo[item.key]}}</text>
			</view>
		</view>

		<view class="uni-divider">
			<view class="uni-divider__content">位面信息</view>
			<view class="uni-divider__line"></view>
		</view>

		<view class="attr-flex">
			<view v-for="(item) in viewConfig.weimianInfoConfig" :key="item.key" :class="getAttrClass(item.columnSize)">
				<text>{{ item.label }}</text>
				<text>{{ roleInfo[item.key]}}</text>
			</view>
		</view> -->
		
		<view class="uni-divider">
			<view class="uni-divider__content">挂机设置</view>
			<view class="uni-divider__line"></view>
		</view>
		
		<view class="uni-list-no-border">
				<view class="uni-list-cell uni-list-cell-pd-mini">
		      <view class="uni-list-cell-db">领取离线倍数</view>
		      <radio-group @change="radioChange" class="flex-lixian-item radio-flex">
			      <label class="radio-flex-item" v-for="(item, index) in lixianbeishuList" :key="item.value">
			        <radio :value="item.value" :checked="index === currentLilianebishu" />{{item.name}}
			      </label>
			    </radio-group>
		    </view>
				<view v-for="(item) in viewConfig.switchConfig" :key="item.key" class="uni-list-cell uni-list-cell-pd-mini">
		        <view class="uni-list-cell-db">{{item.label}}</view>
		        <switch :checked="!!configInfo[item.key]" @change="changeSwitchBoolean(item.key)"/>
		    </view>


				<view v-for="(item) in viewConfig.pickerSwitchConfig" :key="item.key" class="uni-list-cell-no-border uni-list-cell-pd-mini">
					<view class="flex-item-two">
							<view class="uni-list-cell-db">
									<picker @change="changePickerConfig($event, item.key)" :value="configInfo[item.key]" class="background-picker" range-key="text" :range="options[item.key]">
											<view class="uni-input">{{options[item.key][configInfo[item.key]].text}}</view>
									</picker>
							</view>
					</view>
					<view class="flex-item-two">
						<view class="uni-list-cell-db">{{item.label}}</view>
		        <switch :checked="!!configInfo[item.key]" @change="changePickerSwith($event,item.key)"/>
					</view>
		    </view>

		</view>

		<view class="uni-divider">
			<view class="uni-divider__content">功法设置</view>
			<view class="uni-divider__line"></view>
		</view>

		<view class="gongfa-wrap">
			<view v-for="(item) in viewConfig.gongfaConfig" :key="item.key" class="list-cell flex-item-two">
		    <view class="gongfa-list-left">{{ item.label }}</view>
		    <view class="uni-list-cell-db">
		      <picker @change="changeGongfa($event, item.key)" :value="gongfaIndex[item.key]" class="bg-picker-gongfa"  range-key="text" :range="options.gongfa">
		          <view :class="{ active: gongfaIndex[item.key] !== 0 }" class="uni-input">{{options.gongfa[gongfaIndex[item.key]].text}}</view>
		      </picker>
		    </view>
		  </view>
		</view>

		<view class="save-btn">
			<button type="primary" size="mini" @tap="handleChangeConfigInfo">保存设置</button>
		</view>	
	</view>
</template>

<script>
import CryptoJS from 'crypto-js'
import save from '@/utils/save'
import moment from 'moment'
import {mapState,mapMutations} from 'vuex'
import { getValueByIndex, getIndexByValue, getChannel, toast, formatServerList, formatLastServerList } from '@/utils/index'
import { startGuaji, stopGuaji } from '@/api/game'
import { getRoleInfo, getConfigInfo, changeConfigInfo, getUtils, loginFuzhu } from '@/api/game'
import mInput from '../../components/m-input.vue'

export default {
	components:{
		mInput
	},
	filters: {
    statusFilter(status) {
      const statusMap = {
        0: '关闭',
        1: '开启'
      }
      return statusMap[status]
    },
    vipStatus(isVip) {
      const map = {
        0: '普通版',
        1: 'VIP版'
      }
      return map[isVip]
    },
    // 单位换算
    valueFormatFilter(str) {
			if (isNaN(str)) return str
      const numVal = Number(str)
			if (isNaN(numVal)) return ''
			if (numVal < 0) return '未获取到'
      if (numVal > 100000000) {
        return (numVal / 100000000).toFixed(1) + '亿'
      } else if (numVal > 1000000) {
        return (numVal / 10000).toFixed(1) + '万'
      } else {
				return str
			}
    }
  },
	data() {
		return {
			saleChannel: '',
			platformName: '',
			serverName: '',
			lastServerIndex: 0,
			allServerindex: 0,
			utils: {},
			index: 0,
			current: 0,
			statusLoading: false,
			yunguaji: false,
			viewConfig: {},
			configInfo: {},
			options: {},
			gongfaObj: {}, // 功法购买相关的配置
			roleInfo: {},
			gongfaIndex: {},
			gongfaKey: [],
			flag: {
			    loginFlag: false,
			    logoutFlag: false,
			    newUserFlag: false,
			    showServer: false,
					saveRoleFlag: false
			},
			fuzuStatus: {
        end_time: '',
        on_off: 0,
        isExpired: false
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
				sessionid: '',
				userId: '',
				uid: '', // 渠道登录的时候uid和userId不同
				token: '',
				channelId: '',
				pfId: '',
				time: ''
			},
			serverInfo: { // 服务器列表
				client_ip: '',
				server_list: [],
				last_server_list: []
			},
			lixianbeishuList: [
				{value: '1', name: '一倍'},
				{value: '2', name: '两倍'},
				{value: '5', name: '五倍'}
			],
			roleList: [],
			roleIndex: 0,
			selectRoleName: '',
			autocompleteStringList: []
		}
	},
	computed: {
    isPassedTwoHours() {
      const a = moment(new Date())
      const b = moment(this.roleInfo.update_time)
      const duration = a.diff(b)
      return duration > 2 * 3600 * 1000
    },
    // 计算位面位置是否未占有
    isNoWeimian() {
      return this.roleInfo.weimian_weizhi === 0
    },

    // 是否获取到充值额度
    notGetChargeValue() {
      return this.roleInfo.charge_value === -1
    },
		
		// 当前选中的倍数index
		currentLilianebishu() {
			return this.lixianbeishuList.findIndex(item => {
				return Number(item.value) === this.configInfo.lixianbeishu
			})
		}
	},
	onLoad() {
		this.viewConfig = getApp().globalData.viewConfig
		this.configInfo = Object.assign({}, getApp().globalData.viewConfig.configInfoDefault)
		this.roleInfo = Object.assign({}, getApp().globalData.viewConfig.roleInfoDefault)
		this.gongfaIndex = Object.assign({}, getApp().globalData.viewConfig.gongfaIndex)
		this.options = getApp().globalData.viewConfig.options
		this.initGongfaConfig()
		this.saleChannel = getChannel()
		this.handleGetUtils()
		this.loadLoginInfo()
	},
	methods: {
		handleLogin() {
			uni.reLaunch({
			    url: '../login/login'
			})
		},
		loginSwitch() {
			uni.navigateTo({
			    url: '../login/login'
			})
		},

		transferTime() {
			uni.navigateTo({
			    url: '../transfer/transfer'
			})
		},

		handleGetUtils() {
      getUtils().then(res => {
				this.utils = res
				if (this.$global.fuzhuVersion < res.version) {
					uni.showTabBarRedDot({
						index: 3
					})
				}
      }).catch(err => {
        console.log(err)
      })
		},

		// 收藏角色到本地
		handleSaveRole() {
			if (!this.flag.saveRoleFlag) {
				toast('角色信息错误，收藏失败！')
				return
			}
			const saveRoleObj = {
				roleName: this.roleInfo.role_name + '-' + this.serverName,
				userId: this.userInfo.userId,
				platformName: this.platformName,
				loginType: this.userInfo.loginType,
				serverId: this.userInfo.server,
				serverName: this.serverName,
				passwordPlatForm: this.userInfo.passwordPlatForm
			}
			const indexRole = this.roleList.findIndex((item) => {
				return item.roleName === saveRoleObj.roleName
			})
			if (indexRole === -1) {
				this.roleList.unshift(saveRoleObj)
			} else {
				this.roleList[indexRole] = saveRoleObj
			}
			this.saveRoleInfo()
			uni.showToast({
				title: '收藏成功',
				duration: 2000,
				icon: 'none'
			})
		},

		// 删除收藏的角色
		handleRemoveRole() {
			const roleName = this.roleInfo.role_name + '-' + this.serverName
			const indexRole = this.roleList.findIndex((item) => {
				return item.roleName === roleName
			})
			if (indexRole === -1) {
				uni.showToast({
					title: '收藏列表无该角色',
					duration: 2000,
					icon: 'none'
				})
			} else {
				this.roleList.splice(indexRole, 1)
				this.saveRoleInfo()
				uni.showToast({
					title: '移除成功',
					duration: 2000,
					icon: 'none'
				})
			}
		},

		// 清空收藏角色
		handleClearSaveRole() {
			this.roleList = []
			this.saveRoleInfo()
			uni.showToast({
				title: '清空成功',
				duration: 2000,
				icon: 'none'
			})
		},

		// 初始化功法配置
		initGongfaConfig() {
			const gongfaConfig = getApp().globalData.viewConfig.gongfaConfig
			if (gongfaConfig.length > 0) {
				gongfaConfig.forEach(item => {
					this.gongfaKey.push(item.key)
				})
			}
		},

		// 读取记住的登录信息
		loadLoginInfo() {
			uni.setNavigationBarTitle({
					title: '极道仙尊火箭辅助V' + this.$global.fuzhuVersionName
			});
			this.roleList = save.getRoleList()
			const gameLoginInfo = save.getGameLoginInfo()
			if (gameLoginInfo.serverInfo) {
				this.userInfo.platform = gameLoginInfo.platform
				this.userInfo.server = gameLoginInfo.server
				this.userInfo.usernamePlatForm = gameLoginInfo.usernamePlatForm
				this.userInfo.passwordPlatForm = gameLoginInfo.passwordPlatForm
				this.userInfo.username = gameLoginInfo.username
				this.userInfo.uid = gameLoginInfo.uid
				this.userInfo.sessionid = gameLoginInfo.sessionid
				this.userInfo.loginType = gameLoginInfo.loginType
				this.userInfo.userId = gameLoginInfo.userId
				this.platformName = gameLoginInfo.platformName
				this.flag.showServer = gameLoginInfo.showServer
				this.serverInfo = gameLoginInfo.serverInfo
				if (Array.isArray(gameLoginInfo.autocompleteStringList)) this.autocompleteStringList = gameLoginInfo.autocompleteStringList
				this.initSaveData()
				this.handleGuajiStatus()
			}
		},
		// 存储登录信息到LocalStorage
		saveLoginInfo() {
			const gameLoginInfo = {
				platform: this.userInfo.platform,
				server: this.userInfo.server,
				usernamePlatForm: this.userInfo.usernamePlatForm,
				passwordPlatForm: this.userInfo.passwordPlatForm,
				uid: this.userInfo.uid,
				sessionid: this.userInfo.sessionid,
				loginType: this.userInfo.loginType,
				username: this.userInfo.username,
				userId: this.userInfo.userId,
				showServer: this.flag.showServer,
				platformName: this.platformName,
				serverInfo: this.serverInfo,
				autocompleteStringList: this.autocompleteStringList
			}
			save.setGameLoginInfo(gameLoginInfo)
		},

		// 存储收藏的角色
		saveRoleInfo() {
			save.setRoleList(this.roleList)
		},

		// 加载后将存储的数据显示出来
		initSaveData() {
			const lastServerIndex = getIndexByValue(this.serverInfo.last_server_list, this.userInfo.server)
			if (lastServerIndex !== -1) {
				this.serverName = this.serverInfo.last_server_list[lastServerIndex].text
				this.lastServerIndex = lastServerIndex
			} else {
				this.serverName = ''
			}
		},

		// 选择保存的角色
		changeSavedRoleList(e) {
			this.selectRoleName = this.roleList[e.target.value]['roleName']
			this.userInfo.userId = this.roleList[e.target.value]['userId']
			this.userInfo.server = this.roleList[e.target.value]['serverId']
			this.userInfo.loginType = this.roleList[e.target.value]['loginType']
			this.platformName = this.roleList[e.target.value]['platformName']
			this.userInfo.passwordPlatForm = this.roleList[e.target.value]['passwordPlatForm']
			this.handleGetServerList()
			this.handleGuajiStatus()
		},

		// 选择最后登录服务器
		changeLastServer: function(e) {
			this.lastServerIndex = e.target.value
			this.serverName = this.serverInfo.last_server_list[this.lastServerIndex].text
			this.userInfo.server = getValueByIndex(this.serverInfo.last_server_list, e.target.value)
			this.allServerindex = getIndexByValue(this.serverInfo.server_list, this.userInfo.server)
			this.handleGuajiStatus()
		},
		// 选择所以登录服务器
		changeAllServer: function(e) {
			this.userInfo.server = getValueByIndex(this.serverInfo.server_list, e.target.value)
			this.allServerindex = e.target.value
			this.lastServerIndex = getIndexByValue(this.serverInfo.last_server_list, this.userInfo.server)
			this.saveLoginInfo()
		},

		// 更新服务器列表
    handleGetServerList() {
      const params = {
				uname: this.userInfo.usernamePlatForm,
				upwd: this.userInfo.passwordPlatForm,
				login_type: this.userInfo.loginType,
				youxi_name: 'jdxz'
			}
			loginFuzhu(params).then(res => {
				if (res.code === 200) {
					this.serverInfo.server_list = formatServerList(res.data.server_info.server_list)
					this.serverInfo.last_server_list = formatLastServerList(res.data.server_info.last_server_list)
					this.saveLoginInfo()
					uni.showToast({
						title: '服务器更新成功',
						duration: 2000,
						icon: 'none'
					})
				} else if (res.code === 403) {
					uni.showToast({
						title: '更新失败',
						duration: 2000,
						icon: 'none'
					})
				}
			}).catch(err => {
        console.log(err)
      })
		},
		
		// 获取挂机状态
    handleGuajiStatus() {
      if (!this.userInfo.userId) {
				uni.showToast({
					title: '没有获取到登录的用户信息，请重新登录。',
					duration: 2000,
					icon: 'none'
				})
        return
			}
			if (!this.userInfo.server) {
				uni.showModal({
					title: '提示',
					content: '请选择服务器',
					showCancel: false,
					confirmText: '好的',
					success: function (res) {
						if (res.confirm) {
							console.log('用户点击确定');
						} else if (res.cancel) {
							console.log('用户点击取消');
						}
					}
				})
        return
			}
      const param = {
        userid: this.userInfo.userId,
				server_id: this.userInfo.server,
				t: new Date().getTime()
			}
			this.statusLoading = true
      getRoleInfo(param).then(res => { // 查询角色信息
				const code = res.code
				this.statusLoading = false
        switch (code) {
          case 200:
						this.roleInfo = res.data
						this.flag.saveRoleFlag = true
						this.calsIsExpired(res.data.end_time)
						this.saveLoginInfo()
            this.yunguaji = true
						uni.showToast({
							title: '查询挂机状态成功',
							duration: 2000,
							icon: 'none'
						})
            break
          case 403:
						this.flag.saveRoleFlag = false
						uni.showToast({
							title: '参数错误',
							duration: 2000,
							icon: 'none'
						})
            break
          case 404:
						this.flag.saveRoleFlag = false
						this.yunguaji = false
						this.roleInfo = Object.assign({}, getApp().globalData.viewConfig.roleInfoDefault)
						uni.showToast({
							title: '未查询到挂机信息，请开启云挂机',
							duration: 2000,
							icon: 'none'
						})
            break
        }
      }).catch(err => {
				this.flag.saveRoleFlag = false
				this.statusLoading = false
				toast("查询失败，请稍后重试")
        console.log(err)
      })
      this.handleGetConfigInfo()
    },

    // 查询配置信息
    handleGetConfigInfo() {
      if (!this.userInfo.userId || !this.userInfo.server) {
				uni.showToast({
					title: '没有获取到登录的用户信息，请重新登录',
					duration: 2000,
					icon: 'none'
				})
        return
      }
      const param = {
        userid: this.userInfo.userId,
				server_id: this.userInfo.server,
				t: new Date().getTime()
      }
      getConfigInfo(param).then(res => {
        const code = res.code
        switch (code) {
          case 200:
            this.isClickLilianbeishu = false
						this.configInfo = this.correctSetting(res.data)
						this.calcGongfagoumai(String(res.data.gongfagoumai))
            break
          case 403:
						uni.showToast({
							title: '参数错误',
							duration: 2000,
							icon: 'none'
						})
            break
          case 404:
            this.yunguaji = false
						uni.showToast({
							title: '未查询到挂机信息，请开启云挂机',
							duration: 2000,
							icon: 'none'
						})
            this.configInfo = Object.assign({}, this.viewConfig.configInfoDefault)
            break
        }
      }).catch(err => {
        console.log(err)
      })
		},

		// 将后台没返回的设置想设置默认值
		correctSetting(configInfoBackend) {
			const configInfo = Object.assign({}, this.viewConfig.configInfoDefault)
      for (const key in configInfo) {
        if (configInfoBackend.hasOwnProperty(key)) {
          configInfo[key] = configInfoBackend[key]
        }
      }
      return configInfo
		},
		
		// 修改配置项是0和1的情况
		changeSwitchBoolean(item) {
			if (this.configInfo[item]) {
				this.configInfo[item] = 0
			} else {
				this.configInfo[item] = 1
			}
		},

		// 改变picker选项
		changePickerConfig(e, configKey) {
			const index = e.target.value
			this.configInfo[configKey] = index
		},

		// 修改下拉选项后面的开关
		changePickerSwith(e, configKey) {
			const checked = e.target.value
			if (!checked) {
				this.configInfo[configKey] = 0
			} else {
				this.$toast('请选择左侧列表中选项')
			}
		},

		// 改变功法相关的picker
		changeGongfa(e, gongfaKey) {
			const index = e.target.value
			this.gongfaIndex[gongfaKey] = index
		},

    /**
     * 计算功法购买的各种属性
     * 总共12为数字每两位表示一个属性，从前到后是攻击，生命，物防，法防，残页，绝学
     * 两位数字是10-22之间，表示购买不同价格的功法
     */
    calcGongfagoumai(cfgStr) {
      // if (cfgStr.length < 12) {
      //   uni.showModal({
      //     title: '提示',
			// 		content: '功法购买功能有变更，请重新设置后保存！',
			// 		success: function (res) {
			// 			if (res.confirm) {
			// 				console.log('用户点击确定');
			// 			} else if (res.cancel) {
			// 				console.log('用户点击取消');
			// 			}
			// 		}
      //   })
      //   return
			// }
			if (isNaN(cfgStr)) cfgStr = "101010101010"
			this.gongfaKey.forEach((item, index) => {
				const gf = cfgStr.slice(index * 2, 2 + index * 2)
				this.gongfaIndex[item] = parseInt(gf) - 10
			})
    },

    // 从购买功法的对象中生成购买功法的配置数据发到后端
    genGongfagoumaiCfg() {
			let gongfaCalc = ''
			this.gongfaKey.forEach(item => {
				const gfstr = String(this.gongfaIndex[item] + 10)
				gongfaCalc = gongfaCalc + gfstr
			})
			return gongfaCalc
    },

    // 计算辅助到期时间
    calsIsExpired(endTime) {
      const isExpired = moment(endTime).isBefore(new Date())
      if (isExpired) {
        this.fuzuStatus.isExpired = true
        this.fuzuStatus.end_time = '已经过期，请购买！'
      } else {
        this.fuzuStatus.isExpired = false
        this.fuzuStatus.end_time = endTime
      }
    },

    // 修改配置信息
    handleChangeConfigInfo() {
      this.configInfo.gongfagoumai = this.genGongfagoumaiCfg()
      const param = {
        userid: this.userInfo.userId,
        server_id: this.userInfo.server,
        pwd_md5: CryptoJS.MD5(this.userInfo.passwordPlatForm).toString(),
        setting_content: this.configInfo
      }
      changeConfigInfo(param).then(res => {
        const code = res.code
        switch (code) {
          case 200:
            this.handleGetConfigInfo()
						uni.showToast({
							title: '修改成功',
							duration: 2000,
							icon: 'none'
						})
            break
          case 403:
						uni.showToast({
							title: '参数错误',
							duration: 2000,
							icon: 'none'
						})
            break
          case 404:
            this.yunguaji = false
						uni.showToast({
							title: '未查询到挂机信息，请开启云挂机',
							duration: 2000,
							icon: 'none'
						})
            break
        }
      }).catch(err => {
        console.log(err)
      })
    },

    // 开始云挂机
    handleStartguaji() {
      if (!this.userInfo.userId || !this.userInfo.server) {
				uni.showToast({
					title: '没有获取到登录的用户信息，请重新登录',
					duration: 2000,
					icon: 'none'
				})
        return
      }
      const param = {
        userid: this.userInfo.userId,
        server_id: this.userInfo.server,
        pwd_md5: CryptoJS.MD5(this.userInfo.passwordPlatForm).toString()
      }
      startGuaji(param).then(res => {
        const code = res.code
        switch (code) {
          case 200:
						uni.showToast({
							title: '挂机成功',
							duration: 2000,
							icon: 'none'
						})
						this.handleGuajiStatus()
            break
          case 300:
						uni.showToast({
							title: '新用户挂机成功，请15秒后查询挂机状态',
							duration: 2000,
							icon: 'none'
						})
						const self = this
						setTimeout(function() {
							self.handleGuajiStatus()
						}, 15000)
            break
          case 403:
						uni.showToast({
							title: '挂机失败，请重新登录后重试',
							duration: 2000,
							icon: 'none'
						})
            break
        }
      })
    },

    // 停止云挂机
    handleStopguaji() {
      const param = {
        userid: this.userInfo.userId,
        server_id: this.userInfo.server,
        pwd_md5: CryptoJS.MD5(this.userInfo.passwordPlatForm).toString()
      }
      stopGuaji(param).then(res => {
        const code = res.code
        switch (code) {
          case 200:
						uni.showToast({
							title: '停止挂机成功',
							duration: 2000,
							icon: 'none'
						})
						this.handleGuajiStatus()
            break
          case 403:
						uni.showToast({
							title: '操作失败',
							duration: 2000,
							icon: 'none'
						})
            break
          case 404:
						uni.showToast({
							title: '操作失败',
							duration: 2000,
							icon: 'none'
						})
            break
        }
      })
		},

		doCopy() {
			if (!this.userInfo.server || !this.userInfo.userId) {
				toast("复制失败,没有选择服务器或者没有登录")
				return
			}
			const cpText = `服务器:${this.userInfo.server},续费ID:${this.userInfo.userId},游戏:fsxl`
			// #ifdef APP-PLUS
			uni.setClipboardData({
				data: String(cpText),
				success: function () {
					toast("复制成功,可用于辅助续费充值")
				}
			})
			// #endif
			// #ifdef H5
			this.$copyText(cpText).then(
        res => {
          toast("复制成功,可用于辅助续费充值")
        },
        err => {
          toast("复制失败")
        }
			)
			// #endif
		},

		radioChange: function(evt) {
			this.configInfo.lixianbeishu = Number(evt.target.value)
			switch (Number(evt.target.value)) {
				case 1:
					uni.showToast({
						title: '选择一倍，免费',
						duration: 2000,
						icon: 'none'
					})
					break
				case 2:
					uni.showToast({
						title: '选择两倍，要仙缘',
						duration: 2000,
						icon: 'none'
					})
					break
				case 5:
					uni.showToast({
						title: '选择五倍，要仙缘',
						duration: 2000,
						icon: 'none'
					})
          break
			}
		  // for (let i = 0; i < this.lixianbeishuList.length; i++) {
		  //     if (this.lixianbeishuList[i].value === evt.target.value) {
		  //         this.current = i;
		  //         break;
		  //     }
		  // }
		},

		getAttrClass(columnSize) {
			if (columnSize === 1) {
				return 'attr-flex-item-1'
			} else if (columnSize === 2) {
				return 'attr-flex-item-2'
			} else {
				return 'attr-flex-item-3'
			}
		}
	}
}
</script>

<style lang="css">
	.background-picker .uni-input{
		background: #ebedf0 !important;
		padding: 5upx 25upx;
		overflow: hidden;
	}
	.bg-picker-gongfa .uni-input{
		background: #ebedf0 !important;
		padding: 5upx 10upx;
		margin: 10upx;
		overflow: hidden;
	}
</style>
<style lang="css" scoped>
.gongfa-wrap {
	background: #ffffff;
	display: flex;
	flex-wrap: wrap;
	align-items: center;
	justify-content: space-between;
}
.gongfa-list-left {
	font-size:28upx;
}
.active {
	color: #1989fa;
}
.btn-center {
	display: flex;
	align-items: center;
	justify-content: center;
}
.btn-center-margin {
	margin-top: 20upx;
}
.attr-flex {
	width: 100%;
	display: flex;
	flex-wrap: wrap;
	justify-content: space-between;
}
.attr-flex-item-3 {
	width: 33.3%;
	/* 加入这两个后每个item的宽度就生效了 */
	min-width: 33.3%;
	max-width: 33.3%;
}
.attr-flex-item-2 {
	width: 48%;
	/* 加入这两个后每个item的宽度就生效了 */
	min-width: 48%;
	max-width: 48%;
}
.attr-flex-item-1 {
	width: 100%;
}
.flex-item-two {
	display: flex;
	width: 48%;
	min-width: 48%;
	max-width: 48%;
}
.flex-lixian-item {
	flex: 1;
	flex-basis: auto;
	width: 65%;
	min-width: 65%;
	max-width: 65%;
}
.list-cell {
	display: flex;
	flex: 1;
	flex-basis: auto;
	flex-direction: row;
	justify-content: space-between;
	align-items: center;
}
.content .uni-list-cell-left {
	width: 35%;
	text-align: right;
}
.radio-flex {
	display: flex;
}
.radio-flex-item {
	flex: 1;
	flex-basis: auto;
	width: 32%;
	/* 加入这两个后每个item的宽度就生效了 */
	min-width: 32%;
	max-width: 32%;
}
.uni-list-no-border {
	background-color: #FFFFFF;
	position: relative;
	width: 100%;
	display: flex;
	flex-direction: column;
}
.uni-list-cell-no-border {
	position: relative;
	display: flex;
	flex-direction: row;
	justify-content: space-between;
	align-items: center;
}
.uni-list-cell-pd-mini {
	padding: 5upx 30upx 5upx 10upx;
}
.danger {
  color: red;
}
.save-btn{
	margin: 50upx;
	text-align: center;
}
.waring-wrap {
  color: red;
  margin-top: 10upx;
  white-space: pre-line;
	user-select: text;
}
.content-wrap {
	user-select: text;
}
.fuzhu-info {
	display: flex;
	align-items: center;
}
.copy-button {
	margin-left:40upx;
}
</style>
