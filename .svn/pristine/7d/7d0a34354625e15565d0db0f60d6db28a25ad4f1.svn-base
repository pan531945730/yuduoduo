<template>
	<div class="online-pay" :class="{'more-pay-bg': (payType === 'more')}">
		<div class="first-pay" v-if="payType === 'first'">
			<template v-if="!lastPay">
				<div class="tip" v-if="hasFront">您可选择支付房租或定金</div>
				<div v-if="hasRental" class="pay-box" @click="payChange('rental', rental, payTypes[4].id + payTypes[8].id )">
					<div class="pay-box-con">
						<i class="rental"></i>
						<div class="money">
							<p class="price">￥<span class="number">{{ rental }}</span></p>
							<p class="text">{{ rentalText }}</p>
						</div>
					</div>
					<div class="pay-box-tip">
						缴纳房租后，请于起租日入住哦
					</div>
					<em class="selected" v-show="paySelected === 'rental'"></em>
				</div>
				<div v-if="hasFront" class="pay-box" @click="payChange('front', front, payTypes[3].id)">
					<div class="pay-box-con">
						<i class="front"></i>
						<div class="money">
							<p class="price">￥<span class="number">{{ front }}</span></p>
							<p class="text">需交定金</p>
						</div>
					</div>
					<div class="pay-box-tip">
						请于起租日前付清房租尾款<span class="point">{{ lastSum }}元</span>
					</div>
					<em class="selected" v-show="paySelected === 'front'"></em>
				</div>
			</template>
			<template v-if="lastPay">
				<div class="pay-box">
					<div class="pay-box-con">
						<i class="front"></i>
						<div class="money">
							<p class="price">￥<span class="number">{{ lastSum }}</span></p>
							<p class="text">房租尾款</p>
						</div>
					</div>
					<div class="pay-box-tip">
						已付定金<span class="point">{{ front }}元</span>，请于起租日前支付房租尾款
					</div>
				</div>
			</template>
			<pay-btn :payData="payData" :amount="paySum" :payType="payId"
			         className="long-btn pay-btn" text="确认支付"
			         @pay-success="paySuccess"
			>

			</pay-btn>
		</div><!--first-pay-->
		<div class="more-pay" v-if="payType === 'more'">
			<v-header title="在线支付" menuText="历史缴费" @on-menu="goHistory" :hideBack="true">
			</v-header>
			<div class="list">
				<ul>
					<li class="list-item" v-for="(item, index) in listData">
						<div class="content">
							<p class="title">{{ item.name }}</p>
							<p class="tip">{{ item.startDate }}-{{ item.endDate }}</p>
						</div>
						<span class="money">{{ item.price }}元</span>
						<pay-btn :payData="payData" :amount="item.price" :payType="item.payId" @pay-success="paySuccess('list')"></pay-btn>
					</li>
				</ul>
			</div>
			<div class="checkout" v-show="!payEmpty">
				<div class="total">
					合计：
					<span class="number">￥{{ totalSum }}</span>
				</div>
				<pay-btn :payData="payData" :amount="totalSum" :payType="totalId" text="结 算" @pay-success="paySuccess('list')"></pay-btn>
			</div>
		</div><!--more-pay-->
		<div class="pay-empty" v-show="payEmpty">
			<div class="box">
				<i></i>
				<p>暂无待缴费记录</p>
			</div>
		</div>
	</div>
</template>

<script>
import VHeader from '../components/Header'
import PayBtn from '../components/PayBtn'
import { getUrlParams, setUrlParams, ss} from '../common/util'
import { appid, payType, aliJs } from '../common/config'
import { payList, wxSignature, wxOpenid, createOrder, wxPay, contractList } from '../common/api'
var code = getUrlParams('code', window.location.href, 0)
var oldCode = ss.get('code')
var payTypes = {}
payType.forEach(item => {
    payTypes[item.id] = item.name
})
export default {
    components: {
        VHeader,
        PayBtn
    },
    data () {
        return {
            payTypes: payType,
            payType: '', //first首次支付房租 more非首次
	        hasRental: true, //是否有房租账单
	        hasFront: true, //是否有定金账单
            lastPay: false, //尾款支付
	        paySum: 0, //首次支付金额
            payId: 0, //房租支付类型
            paySelected: 'rental', //选择项 front订金 rental租金
            front: 0, //订金
            rental: 0, //租金
	        rentalText: '', //房租提示词
	        lastSum: 0, //尾款金额


            totalSum: 0, //非首次支付总额
            totalId: 0, //支付总额Id
            listData: [], //列表数据
	        url: window.location.href,
            payEmpty: false, //是否有待支付账单
            payData: { //支付基础数据
                houseId: 0, //房源ID
                roomId: 0, //房间ID
                userId: 0, //用户ID
            }
        }
    },
    created() {
		this.setTitle('在线支付')
	    this.userId = this.isLogin()
	    this.conId = getUrlParams('conId', this.url)
        this.houseId = getUrlParams('houseId', this.url)
        this.roomId = getUrlParams('roomId', this.url)
	    if (!this.userId) {
		    this.goLogin()
			return
	    }
	    if (!this.conId) {
            this.hideLoading()
            this.payEmpty = true
		    return
	    }

        this.payData = {
            houseId: this.houseId, //房源ID
            roomId: this.roomId, //房间ID
            userId: this.userId, //用户ID
        }
	    this.getContract()
		this.aliLoad()
	},
    methods: {
        oauthUrl() {
            var href = 'https://open.weixin.qq.com/connect/oauth2/authorize?appid='+ appid +'&redirect_uri='+  encodeURIComponent(this.url)+'&response_type=code&scope=snsapi_base&state=yudoodoo#wechat_redirect'
            window.location.replace(href)
        },
	    //去签合同
	    goContract() {
            let url = setUrlParams({
                conId: this.conId,
                houseId: this.houseId,
                roomId: this.roomId
            }, 'contract')
			this.$router.push(url)
	    },
        paySuccess(list) { //支付成功
            ss.remove('contractInfo')
			if (this.payId === this.payTypes[7].id + this.payTypes[8].id || list) { //尾款
				this.payType = 'more'
                this.getList()
			} else {
				this.goContract()
			}
        },
        //支付切换
        payChange(type, money, id) {
			this.paySelected = type
	        this.paySum = money
	        this.payId = id
        },
	    //计算总价
        countTotal() {
            let totalSum = 0
	        let totalId = 0
            this.listData.forEach((item) => {
                totalId += item.payId
                totalSum = totalSum + item.price*1000
            })
	        this.totalId = totalId
            this.totalSum = Math.ceil(totalSum)/1000
        },
        goHistory() {
            this.$router.push('historyPay?conId=' + this.conId)
        },
	    //获取合同数据
        getContract() {
            let contractInfo = ss.get('contractInfo')
            try {
                contractInfo = JSON.parse(contractInfo)
            } catch (e) {
                contractInfo = {}
            }
            if (contractInfo.ContractID == this.conId) {
                this.getStats(contractInfo)
				return
            }
            contractList({
                userid: this.userId
            }).then(res => {
                this.hideLoading()
                if (res.ReturnCode == '0' && res.Data != '') {
                    let hasCon = false
                    res.Data.some(item => {
                        if (item.ContractID == this.conId) {
                            hasCon = true
                            this.getStats(item)
                            return true
                        }
                    })
	                if (!hasCon) {
                        this.payEmpty = true
                        this.payType = 'none'
	                }
                } else {
                    this.payEmpty = true
                    this.payType = 'none'
                }
            })
        },
        switchNum(number) {
            const words = ['零', '一', '二', '三', '四', '五', '六', '七', '八', '九', '十']
            number = ~~number
            let str = ''
	        if (number > 10) {
				str = words[~~(number/10)] + '十' + words[number%10]
                str = str.replace('一十', '十')
	        } else {
                str = words[number]
	        }
	        return str
        },
	    //获取支付状态 (首次支付、尾款、签订合同等)
        getStats(contractInfo) {

            /*
            * IsDeposit 是否支付定金，0没付 1已付定金 2已付房租或尾款
            * IsSign 是否签名，0否 1是
            * */
            if (contractInfo.IsDeposit != 0 && contractInfo.IsSign == 0) {
                this.goContract()
				return
            }
			if (contractInfo.IsDeposit == 2) { //非首次支付
                this.payType = 'more'
                this.getList()
				return
			}
            this.hideLoading()
	        //首次支付
            this.payType = 'first'
	        /*
	         * IsDepositBill 是否有定金账单0否1是
	         * IsDepositPay  是否支持付定金 0否1是
	         * IsRentBill  是否有房租账单0否1是
	         * */
            if (contractInfo.IsRentBill == 0) {
                this.hasRental = false
            }
	        if (contractInfo.IsDepositPay == 0 || contractInfo.IsDepositBill == 0) {
                this.hasFront = false
	        }
	        //没付定金且没有定金账单和房租账单
	        if (contractInfo.IsDeposit == 0 && contractInfo.IsDepositBill == 0 && contractInfo.IsRentBill == 0) {
                this.payEmpty = true
		        this.payType = 'none'
		        return
	        }

	        /*
	        * PaymentMonth  付几月
	        * PledgeMonth 押几月
	        * MonthlyRent 每月租金
	        * */
	        this.rentalText = '押'+ this.switchNum(contractInfo.PledgeMonth) +'付' + this.switchNum(contractInfo.PaymentMonth)
	        //总金额
			var total = (contractInfo.PaymentMonth + contractInfo.PledgeMonth) * contractInfo.MonthlyRent || 0
            //订金
            this.front = Math.ceil(contractInfo.MonthlyRent*0.3) || 0
	        // 租金
            this.rental = total
	        //尾款
            this.lastSum = total - this.front
            if (contractInfo.IsDeposit == 0) { //房租
		        this.paySum = total
                this.payId = this.payTypes[4].id + this.payTypes[8].id
	        } else { //尾款
                this.lastPay = true
                this.paySum = this.lastSum
                this.payId = this.payTypes[7].id + this.payTypes[8].id
	        }
        },
	    //获取待付费列表
        getList() {
            /*const listData = [
                {
                    payId: 1,
                    price: 123.343,
                },
                {
                    payId: 2,
                    price: 2323.343,
                },
                {
                    payId: 4,
                    price: 123.343,
                },
                {
                    payId: 32,
                    price: 2323.343,
                }
            ]
            listData.map(item => {
                return item.name = payTypes[item.payId]
            })
		    this.listData = listData
            this.hideLoading()
            this.countTotal()*/

            payList({
                userid: this.userId,
                contractid: this.conId,
	            time: +new Date()
            }).then(res => {
                this.hideLoading()
                if (res.ReturnCode == '0' && res.Data != '') {
                    res.Data.forEach((item, index) => {
						res.Data[index].payId = item.PayType
                        res.Data[index].price = item.ShouldAmount
                        res.Data[index].name = payTypes[item.PayType]
                        res.Data[index].startDate = (item.FinanceStarDate || '')
                        res.Data[index].endDate = (item.FinanceEndDate || '')
                    })
                    this.listData = res.Data
                    this.countTotal()
                } else if (res.ReturnCode == 110) {
					this.payEmpty = true
                } else {
                    this.oauthUrl()
                }
            })
	    },
        //微信签名
        signature(res) {
            /*var data = {}
            try {
                res.signature = JSON.parse(res.signature)
                res.signature.forEach(item => {
                    data[item.Key] = item.Value
                })
                wx.config({
                    debug: false,
                    appId: appid,
                    timestamp: data.timestamp,
                    nonceStr: data.nonceStr,
                    signature: data.signature,
                    jsApiList: ['onMenuShareTimeline', 'chooseWXPay']
                })
                wx.ready(function () {
                    wx.onMenuShareTimeline({
                        title: '寓多多-来了',
                        link: 'http://testapi.yudoodoo.com/wap/',
                        imgUrl: 'http://testapi.yudoodoo.com/wap/static/img/favicon.png',
                        desc: '寓多多-挺好的呀',
                        trigger: function (res) {
                            //alert('用户点击分享到朋友圈');
                        },
                        success: function (res) {
                            //alert('已分享');
                        },
                        cancel: function (res) {
                            //alert('已取消');
                        },
                        fail: function (res) {
                            //alert(JSON.stringify(res));
                        }
                    })
                })
            }catch (e) {
                this.$vux.toast.text('微信签名失败，请重试')
            }*/
        },
        //获取签名数据
	    getSignature() {
            wxSignature({
                url: encodeURIComponent(this.url.split('#')[0])
            }).then(res => {
                if (res.ReturnCode == '0' && res.Data) {
                    this.signature(res.Data)
                } else {
                    this.$vux.toast.text('微信签名失败，请重试')
                }
            })
	    },
        aliLoad() {
            if (window._AP) {
                return
            }
            var script = document.createElement('script')
            script.defer = true
            script.async = true
            script.src = aliJs
            document.body.appendChild(script)
        }
	}
}
</script>

<style lang="scss" scoped>
@import "../assets/css/pages/onlinePay";

</style>
