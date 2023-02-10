<template>
	<!-- http://localhost:6789/douyuxingchen/orderConfirm?goodsId=73 -->
	<div class="root">
		<div class="nav">
			<div class="store" v-if="!$route.query.goods_ids">
				<div class="storeImg"><img :src="goodsData.head_pic[0]" alt=""></div>
				<div class="storedesc">
					<div class="name">{{goodsData.buy_name}}</div>
					<div class="content" v-html="goodsData.intro"></div>
					<div class="price">
						<span class="isusePrice">¥{{goodsData.real_price}}</span>
						<!-- <span class="nousePrice"
							v-if="Number(goodsData.original_price)!==Number(goodsData.real_price)">¥{{goodsData.original_price}}</span> -->
					</div>
				</div>
			</div>
			<div class="goods-diagnose" v-if="$route.query.goods_ids">
				<section class="goods-cell" v-for="(item, index) in diagnoseGoods" :key="index">
					<div class="line-up" :class="{['show-limit']: item.expire_time && item.expire_time > 0}">
						<span class="goods-name">{{item.name}}</span>
						<span
							class="price-now">¥{{item.expire_time && item.expire_time > 0 ? item.real_price : item.original_price}}</span>
						<span class="price-old"
							v-if="item.expire_time && item.expire_time > 0">¥{{item.original_price}}</span>
					</div>
					<div class="line-down" v-if="item.expire_time && item.expire_time > 0">
						<span class="time-tip">距离优惠结束还剩</span>
						<span class="time-left"> {{item.expire_time | format}}</span>
					</div>
					<div class="part-line" :style="diagnoseGoods.length != index + 1 ? 'opacity: 1': 'opacity: 0'">
					</div>
				</section>

			</div>
			<div class="goodsList" :class="{hideBalance: !openBalance}">
				<div class="pricecolum">
					<div class="goodsPrice">
						<span class="text">商品金额</span>
						<span class="val">¥{{priceData.goods_price}}</span>
					</div>
					<div class="goodsPoup" @click="handleAction">
						<span class="text">伴学金</span>
						<span class="val"
							v-if="priceData.estimate_account_price_desc!==''">{{priceData.estimate_account_price_desc}}</span>
						<span class="valcolr" v-else>不可用</span>
						<div class="imgpoup"><img
								src="../../assets/douyuxingchen/storeList/poupopen@2x.png" alt=""></div>
					</div>
					<div class="goodsPoup" v-if="openBalance" @click="handleActionWallet">
						<span class="text">消费金</span>
						<span class="val"
							v-if="priceData.total_balance_desc!==''">{{priceData.total_balance_desc}}</span>
						<span class="valcolr" v-else>不可用</span>
						<div class="imgpoup"><img
								src="../../assets/douyuxingchen/storeList/poupopen@2x.png" alt=""></div>
					</div>
				</div>
				<div class="pricecount">
					<span class="text">合计：<span
							class="val">¥{{((priceData.real_price||0) - 0).toFixed(2)}}</span></span>
				</div>
			</div>

			<div class="beanCake" v-if="equipment.sys ===1 && equipment.app_v >= '1.5.0'">
				<!-- @click="beanCakeSelectPoup(1)" -->
				<div class="beanCake-left">
					<img src="@/assets/douyuxingchen/beanCakeImg.png" alt="">
					<p>豆点余额</p>
				</div>
				<div class="beanCake-right">
					<p>{{priceData.total_doudian}}</p>
					<!-- <div class="beanCake-radio">
                        <van-radio-group v-model="beanCakeRadioPoup"> -->
					<!-- @click="selectPoup(item)" :name="item.id" -->
					<!-- <van-radio icon-size="18px"
                                checked-color="#FB7633"
                                 :name="1"></van-radio>
                        </van-radio-group>
                    </div> -->
				</div>
			</div>
		</div>

		<div class="foot">
			<div class="foot-text" v-if="radioTextShow">
				<div class="textRadio" :class="{clicked: radioText == 1}" @click="handleRadioText">
					<!-- <van-radio-group v-model="radioText">
						<van-radio :name="radiovalue" @click="handleRadioText(radiovalue)" checked-color="#FB7633">
						</van-radio>
					</van-radio-group> -->
					<!-- <div class="checker"></div> -->
					<img class="icon-checker" src="@/assets/wallet/checker.png" alt="">
				</div><span @click="handleRadioText">我已阅读并同意</span><span class="buyText" @click="handleText">《购买须知》</span>
			</div>
			<div class="foot-btn">
				<!-- <span class="left">应付：<span class="payprice">¥{{priceData.real_price}}</span></span> -->
				<span class="left">应付：
					<span class="payprice"
						v-if="equipment.sys ===1 && equipment.app_v >= '1.5.0'">{{(priceData.real_price)}}<span>豆点</span></span>
					<span class="payprice" v-else>¥<font class="bigger">{{(priceData.real_price||0) | dealTextBigger}}</font>
						<font class="smaller">{{(priceData.real_price||0) | dealTextSmaller}}</font>
					</span>
				</span>
				<span class="right" v-if="equipment.sys ===1 && equipment.app_v >= '1.5.0'">
					<!-- todo ：缺少 请充值后购买 逻辑 -->
					<button @click="handleOrderI()" v-if="priceShow">充值后购买</button>
					<button @click="handleOrder()" v-else>立即支付</button>
				</span>
				<span class="right" v-else>
					<button @click="handleOrder()">提交订单</button>
				</span>
			</div>
		</div>
		<div class="poup">
			<van-action-sheet safe-area-inset-bottom v-model="show" title="">
				<div class="sticky-head">
					<div class="header">
						<div class="show" @click="handleuseText">
							<span>使用帮助</span>
							<img src="@/assets/wallet/icon-question.svg" alt="">
						</div>
						<span class="title">伴学金</span>
						<span class="close" @click="handleClose">
							<img src="@/assets/wallet/icon-close.svg" alt="">
						</span>
					</div>
					<div class="tabChenge">
						<van-tabs title-active-color="#AB7FD5" title-inactive-color="#333333" color="#AB7FD5"
							@click="handleTab">
							<van-tab :title="`可用伴学金(${available_count})`"></van-tab>
							<van-tab :title="`不可用伴学金(${unavailable_count})`"></van-tab>
						</van-tabs>
					</div>
				</div>
				<div class="tabContent">
					<div v-if="isUseDiscount===0" class="isUseContent">
						<div v-if="available_count!==0">
							<div class="poupcard" :class="item.locked ? 'disable' : ''" v-for="(item,index) in available_info" :key="index" @click="clickBxj(item)">
								<div class="hedmain">
									<div class="countNumber">
										<span class="countnum">豆伴金账户：<span>{{item.account_no}}</span></span>
										<span class="flright"><span
												class="right">面值：<span>{{item.original_amount}}元</span></span></span>
									</div>
								</div>
								<div class="cardhead">
									<div class="sumtext">余额：<span class="sum">{{item.balance}}</span>元</div>
									<div class="sumradio">
										<!-- todo: -->
										<!-- <van-radio-group v-model="radioPoup">
											<van-radio :ref="`chooser_${item.id}`" @click="selectPoup(item)" :name="item.id"
												checked-color="#FB7633"></van-radio>
										</van-radio-group> -->
										<div class="bxj-checker" :class="bxjRecords.indexOf(item.id) != -1 ? 'clicked' : ''">
											<img class="icon-checker" src="@/assets/wallet/checker.png" alt="">
										</div>
									</div>
								</div>
								<div class="cardmain">
									<div class="countdat">
										<span class="right" v-if="item.expire_day_desc">{{item.expire_day_desc}}</span>
										<span class="datcol"><span>{{item.expire_time_desc}}</span></span>
									</div>
									<div class="countTip">{{item.rule_desc}}</div>
								</div>
							</div>
						</div>
						<div v-else>
							<van-empty description="暂无伴学金" />
						</div>
					</div>
					<div v-if="isUseDiscount===1" class="noUseContent">
						<div v-if="unavailable_count!==0">
							<div class="poupcard" v-for="(item,index) in unavailable_info" :key="index">
								<div class="hedmain">
									<div class="countNumber">
										<span class="countnum">豆伴金账户：<span>{{item.account_no}}</span></span>
										<span class="flright"><span
												class="right">面值：<span>{{item.original_amount}}元</span></span></span>
									</div>
								</div>
								<div class="cardhead">
									<div class="sumtext">余额：<span class="sum">{{item.balance}}</span>元</div>
									<div class="sumradio">
										<img src="../../assets/douyuxingchen/storeList/nousecoin@2x.png" alt="">
									</div>
								</div>
								<div class="cardmain">
									<div class="countdat">有效期：<span>{{item.expire_time_desc}}</span></div>
									<div class="countTip">{{item.rule_desc}}</div>
								</div>
							</div>
						</div>
						<div v-else>
							<van-empty description="暂无伴学金" />
						</div>
					</div>
				</div>
				<div class="sticky-foot">
					<div class="confirmBtn">
						<button @click="handleClose">确定</button>
					</div>
				</div>
			</van-action-sheet>

			<!-- 消费金弹层 -->
			<van-action-sheet safe-area-inset-bottom v-model="showWallet" title="">
				<div class="sticky-head">
					<div class="header">
						<div class="show" @click="handleuseTextWallet">
							<span>使用帮助</span>
							<img src="@/assets/wallet/icon-question.svg" alt="">
						</div>
						<span class="title">消费金</span>
						<span class="close" @click="handleCloseWallet">
							<img src="@/assets/wallet/icon-close.svg" alt="">
						</span>
					</div>
				</div>
				<div class="tabContent is_wallet">
					<div class="isUseContent">
						<div v-if="true">
							<div class="poupcard" @click="clickXfj">
								<div class="cardhead">
									<img src="@/assets/wallet/tiny-line.png" alt="" class="tiny-line">
									<div class="sumtext">余额：<span class="sum">{{priceData.total_balance}}</span>元</div>
									<div class="sumradio">
										<van-radio-group v-model="balance_selected" :disabled="priceData.total_balance_desc === ''">
											<van-radio ref="balancer" :name="balance_selected_value" @click="priceData.total_balance_desc === '' ? ()=>{} : toggleBalance(balance_selected_value)"
												checked-color="#FB7633"></van-radio>
										</van-radio-group>
									</div>
								</div>
								<span class="desc">此消费金可与其它资产叠加使用，提交订单时抵扣商品金额，抵扣后剩余金额将保存至您的消费金中。</span>
							</div>
						</div>
						<div v-else>
							<van-empty description="暂无消费金" />
						</div>
					</div>
				</div>
				<div class="sticky-foot">
					<div class="confirmBtn">
						<button @click="handleCloseWallet">确定</button>
					</div>
				</div>
			</van-action-sheet>

			<!-- 豆点充值弹层 :close-on-click-overlay="false" -->
			<van-action-sheet class="beanCake-action-sheet" safe-area-inset-bottom v-model="beanCakeShowWallet"
				title="">
				<div class="beanCake-sheet-top">
					重要提示
				</div>
				<div class="beanCake-sheet-toptitle">
					iOS系统中购买需使用豆点支付，您当前豆点不足，需充值后购买。
				</div>
				<div class="beanCake-sheet-center">
					<img src="@/assets/douyuxingchen/beanCakeIcon.png" alt="" class="beanCake-sheet-center-img">
					提示：豆点充值后不可退款，不可提现，仅支持iOS设备使用，详细请咨询客服<a style="color: #7dc9a9"
						href="tel:400-6596888">400-6596888</a>。
				</div>
				<div class="beanCake-sheet-btn" @click="beanCakePayment">
					<div>立即充值</div>
				</div>
				<div class="beanCake-sheet-close" @click="beanCakeClose">
					取消
				</div>
			</van-action-sheet>
		</div>
		<div class="main-loading" v-if="loadingFlag">
			<van-loading size="24px" color="#666" vertical></van-loading>
		</div>
	</div>
</template>
<script>
	import {
		mapGetters
	} from 'vuex';
	import Vue from 'vue';
	import {
		ActionSheet,
		Tab,
		Tabs,
		RadioGroup,
		Radio,
		Form,
		Toast
	} from 'vant';
	import {
		getOrderlList,
		getOrderlListDiagnose,
		getReloadprice,
		getordersfirce,
		createDiagnoseOrder
	} from '../../api/douyuxingchen.js'
	import {
		versionJudge
	} from "@/utils/webridge.js"
	import store from '@/store';
	import {
		Empty
	} from 'vant';


	Vue.use(Empty);
	Vue.use(Tab);
	Vue.use(Tabs);
	Vue.use(ActionSheet);
	Vue.use(Radio);
	Vue.use(RadioGroup);
	export default {
		data() {
			return {
				timer_count_down: null,
				beanPayOpened: true, // 豆点开关
				show: false, //是否打开底部弹窗
				radioPoup: 0, //选中的优惠券
				oldradioPoup: 0,
				radioText: 0, //是否同意阅读
				oldradioText: 0,
				radiovalue: 1,
				isUseDiscount: 0, //tab栏当前类型:0适用 1不适用,
				goodsId: 0, //当前商品id,
				goods_ids: "",
				goodsData: {
					head_pic: []
				}, //当前商品展示
				priceData: {}, //当前价格展示
				available_info: [], //可用优惠券展示
				available_count: 0, //可用优惠券数量
				unavailable_info: [], //不可用优惠券展示
				unavailable_count: 0, //不可用优惠券数量
				radioTextShow: true, //购买须知显示
				loadingFlag: false,

				//新增消费金
				showWallet: false, //是否打开底部弹窗
				balance_selected: 0,
				old_balance_selected: 0,
				balance_selected_value: 1,

				// 豆点
				beanCakeRadioPoup: 1,
				beanCakeOldradioPoup: 0,
				beanCakeShowWallet: false,
				priceShow: false,
				availableB: '',
				banPrice: '',
				xfPrice: '',

				default_total_balance_desc: '',
				default_estimate_account_price_desc: '',
				costRecords: [],
				bxjRecords: [],
				openBalance: true,
				kind: 2,
				diagnoseGoods: []
			}
		},
		beforeCreate() {
			document.title = '订单确认'
		},
		computed: {
			...mapGetters(["equipment"]),
		},
		destroyed() {
			clearInterval(this.timer_count_down)
		},
		created() {
			const {
				goods_ids
			} = this.$route.query;
			if (decodeURI(this.getQueryVariable("goodsId"))) {
				this.goodsId = decodeURI(this.getQueryVariable("goodsId"))
				this.getIntroduceList()
			} else if (goods_ids) {
				this.goods_ids = goods_ids;
				this.getIntroduceListDiagnose()
			}
			if (this.$route.query.isbook) {
				this.radioText = 1
				this.radioTextShow = false
			}
		},
		mounted() {
			const {
				goods_ids
			} = this.$route.query;
			window.onRefreshData = () => {
				this.beanCakeShowWallet = false
				let isFromRere = true
				if (decodeURI(this.getQueryVariable("goodsId"))) {
					this.goodsId = decodeURI(this.getQueryVariable("goodsId"))
					this.getIntroduceList(isFromRere)
				} else if (goods_ids) {
					this.goods_ids = this.goods_ids;
					this.getIntroduceListDiagnose(true)
				}
			};
		},
		filters: {
			dealTextBigger(text) {
				text = (text - 0).toFixed(2) + ''
				return text.split('.')[0]
			},
			dealTextSmaller(text) {
				text = (text - 0).toFixed(2) + ''
				return '.' + text.split('.')[1]
			},
			format(value) {
				let second = parseInt(value)
				let minute = 0
				let hour = 0
				let day = 0
				if (second > 60) {
					minute = parseInt(second / 60)
					second = parseInt(second % 60)
					if (minute > 60) {
						hour = parseInt(minute / 60)
						minute = parseInt(minute % 60)
						if (hour > 23) {
							day = parseInt(hour / 24)
							hour = parseInt(hour % 24)
						}
					}
				}

				let result = '' + parseInt(second) + '秒'
				if (minute > 0) {
					result = '' + parseInt(minute) + '分' + result
				}
				if (hour > 0) {
					result = '' + parseInt(hour) + '时' + result
				}
				if (day > 0) {
					result = '' + parseInt(day) + '天'
				}

				return result
			}
		},
		methods: {
			clickXfj(item) {
				let chooser = this.$refs['balancer']
				if (chooser) {
					chooser.$el.click()
				}
			},
			clickBxj(item) {
				if (item.locked) {
					console.log('DEBUG_LOG:lockded abort', item);
					return
				}
				console.log('DEBUG_LOG:click bxj', item);
				//1. 判断bxjRecords是否已经包含此id
				let position = -1;
				this.bxjRecords.map((itemId, index) => {
					if (itemId == item.id) {
						position = index
					}
				})
				if (position != -1) {
					//2. 已经有了，则移除
					this.bxjRecords.splice(position, 1)
				} else {
					//3. 还没有，则加入
					this.bxjRecords.push(item.id)
					//4. 按照即将过期时间，做一次排序，优先使用即将过期的伴学金券
					this.bxjRecords.sort((a, b)=>{
						let result = -1
						let timeA = 0
						let timeB = 0
						this.available_info.map((item, index) => {
							if (item.id == a) {
								timeA = new Date(item.end_time).getTime()
							}
							if (item.id == b) {
								timeB = new Date(item.end_time).getTime()
							}
						})
						result = timeA < timeB ? -1 : 1
						return result
					})
				}

				//5. 遍历操作记录表，并重新记录伴学金的选择状态
				let pos = -1;
				this.costRecords.map((item, index) => {
					if (item.type == 1) {
						pos = index
					}
				})
				if (pos != -1) {
					this.costRecords.splice(pos, 1)
				}
				if (!this.bxjRecords.length) {
					this.availableB = 0
				} else {
					//6. bxjRecords中id，则表示需要遍历累加，计算出已勾选的伴学金总额
					let available = 0;
					this.available_info.map((item, index) => {
						this.bxjRecords.map((itemId, index2) => {
							if (item.id == itemId) {
								available += (Number(item.remain_amount) || 0)
							}
						})
					})
					this.availableB = available
					//7. 由于伴学金的优先级比消费金的优先级高，所以从左插入
					this.costRecords.unshift({
						type: 1,
						total: available	
					})
				}

				this.calcIsRechargeNeeded()
				console.log('DEBUG_LOG:after toggleBxj', this.bxjRecords, this.costRecords);
				this.calcFinalCost()
				this.updateBxjChckersState()
			},
			findBxjItemWithIdAndType(id, type) {
				let result = null
				this.available_info.map((item, index) => {
					if (item.id == id && type == item.used_scope_type) {
						result = item
					}
				})
				return result
			},
			updateBxjChckersState() {
				let goodsCount = Math.max(1, this.diagnoseGoods.length)
				this.available_info.map((item, index) => {
					if (this.bxjRecords.indexOf(item.id) != -1) {
						//已经勾选了的，无需设置禁止点击
					} else {
						let maxCountClickAble;
						if (item.used_scope_type == 1) {
							maxCountClickAble = 1;
						} else if (item.used_scope_type == 1) {
							maxCountClickAble = goodsCount;
						}
						let findedCount = 0
						this.bxjRecords.map((itemId, index2) => {
							if (this.findBxjItemWithIdAndType(itemId, item.used_scope_type)) {
								findedCount++
							}
						})
						if (findedCount >= maxCountClickAble) {
							item.locked = true
						} else {
							item.locked = false
						}
					}
				})
			},
			toggleBalance(item) {
				if (this.selecting) {
					return
				}
				this.selecting = true
				setTimeout(() => {
					this.selecting = false
				}, 150);
				if (item == this.old_balance_selected) {
					this.xfPrice = 0
					this.balance_selected = 0;
					this.old_balance_selected = 0;
				} else {

					let xfPrice = this.priceData.total_balance
					this.xfPrice = xfPrice
					this.old_balance_selected = item
				}


				//total_balance
				let position = -1;
				this.costRecords.map((item, index) => {
					if (item.type == 2) {
						position = index
					}
				})
				if (position != -1) {
					this.costRecords.splice(position, 1)
				}

				if (this.balance_selected == 0) {
					console.log('DEBUG_LOG:取消勾选消费金', this.old_balance_selected);
					this.calcIsRechargeNeeded()
				} else {
					console.log('DEBUG_LOG:勾选消费金', this.old_balance_selected);

					this.costRecords.push({
						type: 2,
						total: this.priceData.total_balance
					})
					this.calcIsRechargeNeeded()
				}
				this.calcFinalCost()
			},
			// selectPoup(item) {
			// 	if (this.selecting) {
			// 		return
			// 	}
			// 	this.selecting = true
			// 	setTimeout(() => {
			// 		this.selecting = false
			// 	}, 150);
			// 	if (item.id == this.oldradioPoup) {
			// 		this.banPrice = 0
			// 		// 取消选中 将 radio、oldRadio的值重置为0。oldRadio 重置是为了防止下次点中与新选中的值相同而判断错误
			// 		this.radioPoup = 0;
			// 		this.oldradioPoup = 0;
			// 		this.chgeReloadprice(this.radioPoup)

			// 	} else {
			// 		this.banPrice = item.balance
			// 		// 选择了新的值，保留此次选择的值，用于下次对比
			// 		this.oldradioPoup = item.id
			// 		this.chgeReloadprice(this.radioPoup)

			// 	}
			// },
			// 计算伴学金 + 余额 + 豆点加起来够不够
			calcIsRechargeNeeded() {
				let myWalletTotal = Number(this.availableB) + Number(this.xfPrice) + Number(this.priceData.total_doudian)
				let goodsPrice = Number(this.priceData.goods_price)
				if (myWalletTotal < goodsPrice) {
					this.priceShow = true
				} else {
					this.priceShow = false
				}
			},
			// 跳转余额页面
			beanCakePayment() {
				webridge.jsToNative({
					handler: 'Common',
					action: 'exeRouter',
					params: {
						url: 'dyxc://com.dbj.app/view/in/mine/doudian',
					},
				})
				// this.beanCakeShowWallet = false
			},
			// 豆点余额点击 -> 目前暂不使用
			// beanCakeSelectPoup(item){
			//     if (item == this.beanCakeOldradioPoup) {
			//         console.log('item----111',item);
			// 		// 取消选中 将 radio、oldRadio的值重置为0。oldRadio 重置是为了防止下次点中与新选中的值相同而判断错误
			// 		this.beanCakeRadioPoup = 0;
			// 		this.beanCakeOldradioPoup = 0;
			//         console.log('this.beanCakeRadioPoup',this.beanCakeRadioPoup);

			// 		// this.chgeReloadprice(this.beanCakeRadioPoup)
			// 	} else {
			//         console.log('item----222',item);
			//         this.beanCakeRadioPoup = 1;
			//         console.log('this.beanCakeRadioPoup',this.beanCakeRadioPoup);

			// 		// 选择了新的值，保留此次选择的值，用于下次对比
			// 		this.beanCakeOldradioPoup = item
			// 		// this.chgeReloadprice(this.beanCakeRadioPoup)
			// 	}
			// },
			handleRadioText(item) {
				// if (item == this.oldradioText) {
				// 	// 取消选中 将 radio、oldRadio的值重置为0。oldRadio 重置是为了防止下次点中与新选中的值相同而判断错误
				// 	this.radioText = 0;
				// 	this.oldradioText = 0;
				// } else {
				// 	// 选择了新的值，保留此次选择的值，用于下次对比
				// 	this.oldradioText = item
				// }
				this.radioText = this.radioText == 1 ? 0 : 1
			},
			chgeReloadprice(val) {
				//todo:
				// getReloadprice({
				// 	goods_id: this.goodsId,
				// 	account_id: val
				// }).then((res) => {
				// 	console.log('DEBUG_LOG:getReloadprice res', res);
				// 	this.priceData = res
				// })

				let position = -1;
				this.costRecords.map((item, index) => {
					if (item.type == 1) {
						position = index
					}
				})
				if (position != -1) {
					this.costRecords.splice(position, 1)
				}
				if (val != 0) {
					let available = 0;
					this.available_info.map((item, index) => {
						if (item.id == val) {
							available = Number(item.remain_amount) || 0
						}
					})
					this.costRecords.push({
						type: 1,
						total: available
					})
					console.log('DEBUG_LOG:勾选伴学金', available);
					this.availableB = available
					this.calcIsRechargeNeeded()
				} else {
					console.log('DEBUG_LOG:取消勾选伴学金');
					this.availableB = 0
					this.calcIsRechargeNeeded()
				}
				this.calcFinalCost()
			},
			onTotalPriceUpdate(totalPrice = 0) {
				this.priceData.goods_price = totalPrice;
				this.calcIsRechargeNeeded()
				this.calcFinalCost()
			},
			calcFinalCost() {

				let costNeeded = Number(this.priceData.goods_price) || 0;
				console.log('DEBUG_LOG:总需花费', costNeeded);
				console.log('DEBUG_LOG:勾选明细', this.costRecords);

				this.priceData.estimate_account_price_desc = this.default_estimate_account_price_desc;
				this.priceData.total_balance_desc = this.default_total_balance_desc;

				console.log('DEBUG_LOG:this.priceData.real_price', this.priceData.real_price);
				this.priceData.real_price = costNeeded;
				this.costRecords.map((item, index) => {
					if (item.type == 1) {
						console.log('item', item);
						// 伴学金
						// priceData.estimate_account_price_desc
						let total = item.total
						let cost = Math.min(total, this.priceData.real_price)
						this.priceData.estimate_account_price_desc = '-' + cost
						this.priceData.real_price = (this.priceData.real_price - cost).toFixed(2)
						// console.log('DEBUG_LOG:还差金额', this.priceData.real_price);
					} else if (item.type == 2) {
						// 消费金
						let total = item.total
						let cost = Math.min(total, this.priceData.real_price)
						this.priceData.total_balance_desc = '-' + cost;
						if (cost == 0 || Number(this.priceData.goods_price) <= Number(this.availableB)) {
							this.priceData.total_balance_desc = ' ';
						}
						this.priceData.real_price = (this.priceData.real_price - cost).toFixed(2)
						// console.log('DEBUG_LOG:还差金额', this.priceData.real_price);
					} else if (item.type == 3) {
						// 豆点
					}
				})
				if (this.default_total_balance_desc &&  Number(this.priceData.goods_price) <= Number(this.availableB)) {
					this.priceData.total_balance_desc = ' ';
				}

			},
			//当前商品订单
			getIntroduceList(isFromRere) {
				getOrderlList({
					goods_id: this.goodsId
				}).then((res) => {
					console.log('DEBUG_LOG:订单详情', res);
					this.goodsData = res.goods
					this.priceData = res.price

					// todo: remove test code
					// this.priceData.goods_price = 601.00
					// this.priceData.total_balance = 600.99
					// this.priceData.total_balance_desc = '有'

					this.available_info = res.account.available_info
					this.available_count = res.account.available_count
					this.unavailable_info = res.account.unavailable_info
					this.unavailable_count = res.account.unavailable_count

					//Add:
					this.default_total_balance_desc = this.priceData.total_balance_desc
					this.default_estimate_account_price_desc = this.priceData.estimate_account_price_desc

					this.calcIsRechargeNeeded()
					if (isFromRere) {
						this.calcFinalCost()
					}
				})

			},
			getIntroduceListDiagnose(isFromRefresh) {
				getOrderlListDiagnose({
					goods_ids: this.goods_ids
				}).then((res) => {
					console.log('DEBUG_LOG:订单详情', res);
					//todo: remove test code - 【诊练优惠调试代码】
					// res.goodsList[0].expire_time = 86400;
					// res.goodsList[1].expire_time = 10;
					// res.goodsList[1].original_price = 100000;

					this.diagnoseGoods = res.goodsList || []
					this.priceData = res.price || {}

					// todo: remove test code - 【消费金调试代码】
					// this.priceData.goods_price = 601.00
					// this.priceData.total_balance = 600.99
					// this.priceData.total_balance_desc = '有'


					this.available_info = (res.account || {}).available_info || []
					this.available_count = (res.account || {}).available_count || 0
					this.unavailable_info = (res.account || {}).unavailable_info || []
					this.unavailable_count = (res.account || {}).unavailable_count || 0

					//Add:
					this.default_total_balance_desc = this.priceData.total_balance_desc
					this.default_estimate_account_price_desc = this.priceData.estimate_account_price_desc
					//calculate the total cost for these goods.
					this.updateDiagnoseGoodsPrice()

					this.calcIsRechargeNeeded()
					if (isFromRefresh) {
						this.calcFinalCost()
					}
				})
			},
			updateDiagnoseGoodsPrice() {
				let start_time = parseInt(Date.now()/1000)
				this.diagnoseGoods.map((item, index) => {
					if (item.expire_time && item.expire_time > 0) {
						item.org_expire_time = item.expire_time
					}
				})
				let update_price = () => {
					let totalPrice = 0;
					this.diagnoseGoods.map((item, index) => {
						if (item.expire_time && item.expire_time > 0) {
							totalPrice += Number(item.real_price);
						} else {
							totalPrice += Number(item.original_price);
						}
					})
					if (this.priceData.goods_price != totalPrice) {
						this.onTotalPriceUpdate(totalPrice)
					}
				}
				update_price()

				clearInterval(this.timer_count_down)
				this.timer_count_down = setInterval(() => {
					let hasTimer = false
					let current_time = parseInt(Date.now()/1000)
					this.diagnoseGoods.map((item, index) => {
						let time_past = current_time - start_time
						if (item.org_expire_time && item.org_expire_time - time_past > 0) {
							hasTimer = true
							item.expire_time = item.org_expire_time - time_past
							// item.expire_time--;
							// console.log('DEBUG_LOG:剩余倒计时', item.expire_time);
						} else {
							// console.log('DEBUG_LOG:倒计时结束', null);
							item.expire_time = null
						}
					})
					if (!hasTimer) {
						console.log('DEBUG_LOG:全部倒计时结束', null);
						clearInterval(this.timer_count_down)
					}
					update_price()
				}, 1000)
			},
			//获取当前商品id
			getQueryVariable(variable) {
				var query = window.location.search.substring(1);
				var vars = query.split("&");
				for (var i = 0; i < vars.length; i++) {
					var pair = vars[i].split("=");
					if (pair[0] == variable) {
						return pair[1];
					}
				}
				return ('');
			},
			//打开底部弹窗
			handleAction() {
				this.show = true
			},
			handleActionWallet() {
				this.showWallet = true
			},
			//切换优惠券tab
			handleTab(name, title) {
				this.isUseDiscount = name
			},
			handleClose() {
				this.show = false
			},
			handleCloseWallet() {
				this.showWallet = false
			},
			beanCakeClose() {
				this.beanCakeShowWallet = false
			},
			handleText() {
				this.toNext('/agreement/userbuynotice')
			},
			handleuseText() {
				this.toNext('/money-explain')
			},
			handleuseTextWallet() {
				this.toNext('/wallet-desc')
			},
			handleOrderI() {
				if (this.radioText === 1) {
					this.beanCakeShowWallet = true
				} else {
					Toast('请勾选我已阅读并同意《购买须知》')
				}
			},
			handleOrder() {
				let priority = []
				console.log('DEBUG_LOG:this.priceData.real_price', this.priceData.real_price);
				let total_payment = Number(this.priceData.real_price||0).toFixed(2);
				// todo: remove test code
				// if (this.diagnoseGoods && this.diagnoseGoods[0] && /L0/.test(this.diagnoseGoods[0].name)) {
				// 	total_payment = '999.90'
				// }
				this.costRecords.map((item, index) => {
					priority.push(item.type)
				})
				if (this.equipment && this.equipment.sys == 1 && this.beanPayOpened && this.equipment.app_v >= '1.5.0') {
					//IOS使用豆点
					priority.push(3)
				}

				if (this.radioText === 1) {
					// this.loadingFlag = true
					const {
						goods_ids
					} = this.$route.query;
					if (goods_ids) {
						//诊练下单
						createDiagnoseOrder({
							goods_ids,
							priority,
							// accountId: this.radioPoup,
							bxj_records: this.bxjRecords,
							total_payment
						}).then((res) => {
							console.log('DEBUG_LOG:提交订单-诊练', res);
							this.loadingFlag = false
							if (res.price_valid == false) {
								webridge.jsToNative({
									handler: 'Diacrisis',
									action: 'showPop',
									params: {
										type: '3',
										tips: res.text
									},
									callback: (confirmed)=>{
										if (confirmed) {
											//点击了确定
										} else {
											//点击了取消
										}
										// window.location.reload()
										window.onRefreshData && window.onRefreshData()
									}
								});
							} else {
								if (res.isPaid === 1) {
									webridge.jsToNative({
										handler: "Common",
										action: "exeRouter",
										params: {
											url: `dyxc://com.dbj.app/view/in/pay/cashierdesk?target_view=` +
												'result' +
												`&diagnose=1` +
												`&order_id=` +
												res.id +
												`&goods_name=` +
												res.name +
												`&real_price=` +
												res.pay_amount +
												`&success_btn_name=` +
												"去设置训练范围" +
												`&success_btn_router=` +
												"dyxc%3A%2F%2Fcom.dbj.app%2Fview%2Fin%2Fdiacrisis%2FtrainingSetting",
										},
									});
									webridge.jsToNative({
										handler: "Common",
										action: "exeAction",
										params: {
											methodName: "closePage"
										},
									});
								} else if (res.isPaid === 0) {
									webridge.jsToNative({
										handler: "Common",
										action: "exeRouter",
										params: {
											url: `dyxc://com.dbj.app/view/in/pay/cashierdesk?target_view=` +
												'cashier' +
												`&diagnose=1` +
												`&order_id=` +
												res.id +
												`&goods_name=` +
												res.name +
												`&real_price=` +
												res.pay_amount +
												`&success_btn_name=` +
												"去设置训练范围" +
												`&success_btn_router=` +
												"dyxc%3A%2F%2Fcom.dbj.app%2Fview%2Fin%2Fdiacrisis%2FtrainingSetting",
										},
									});
									webridge.jsToNative({
										handler: "Common",
										action: "exeAction",
										params: {
											methodName: "closePage"
										},
									});
								}
							}
						}).catch((err) => {
							console.log('DEBUG_LOG:createDiagnoseOrder err', err);
							this.loadingFlag = false
						})

					} else {
						getordersfirce({
							goodsId: this.goodsId,
							// accountId: this.radioPoup,
							bxj_records: this.bxjRecords,
							priority
						}).then((res) => {
							console.log('DEBUG_LOG:提交订单', res);
							this.loadingFlag = false
							// isPaid = 1 为支付成功
							if (res.isPaid === 1) {
								let btn_name = '去学习';
								let btn_router = 'dyxc%3A%2F%2Fcom.dbj.app%2Fview%2Fin%2Froot%2Fmain%3Findex%3D1'
								if(res.goods_type === 11){
									btn_name = '去查看';
									btn_router='dyxc%3A%2F%2Fcom.dbj.app%2Fview%2Fin%2Froot%2Fmain%3Findex%3D2%26subPage%3DsubAccountManage'
								}
								console.log(`btn_name=${btn_name}`)
								webridge.jsToNative({
									handler: "Common",
									action: "exeRouter",
									params: {
										url: `dyxc://com.dbj.app/view/in/pay/cashierdesk?target_view=` +
											'result' +
											`&order_id=` +
											res.id +
											`&goods_name=` +
											res.name +
											`&real_price=` +
											res.pay_amount +
											`&success_btn_name=` +
											btn_name +
											`&success_btn_router=` +
											btn_router,
									},
								});
								webridge.jsToNative({
									handler: "Common",
									action: "exeAction",
									params: {
										methodName: "closePage"
									},
								});
							} else if (res.isPaid === 0) {
								let btn_name = '去学习';
								let btn_router = 'dyxc%3A%2F%2Fcom.dbj.app%2Fview%2Fin%2Froot%2Fmain%3Findex%3D1'
								if(res.goods_type === 11){
									btn_name = '去查看';
									btn_router='dyxc%3A%2F%2Fcom.dbj.app%2Fview%2Fin%2Froot%2Fmain%3Findex%3D2%26subPage%3DsubAccountManage'
								}
								console.log(`btn_name=${btn_name}`)
								webridge.jsToNative({
									handler: "Common",
									action: "exeRouter",
									params: {
										url: `dyxc://com.dbj.app/view/in/pay/cashierdesk?target_view=` +
											'cashier' +
											`&order_id=` +
											res.id +
											`&goods_name=` +
											res.name +
											`&real_price=` +
											res.pay_amount +
											`&success_btn_name=` +
											btn_name +
											`&success_btn_router=` +
											btn_router,
									},
								});
								webridge.jsToNative({
									handler: "Common",
									action: "exeAction",
									params: {
										methodName: "closePage"
									},
								});
							}
						}).catch((err) => {
							console.log('DEBUG_LOG:getordersfirce err', null);
							this.loadingFlag = false
						})
					}
				} else {
					Toast('请勾选我已阅读并同意《购买须知》')
				}
				/* */
			}
		}
	}
</script>
<style lang="scss" scoped>
	::v-deep .van-popup--safe-area-inset-bottom {
		padding-bottom: 0;
	}

	.root {
		width: 100%;
		min-height: 100vh;
		background: #F5F5F5;
		position: relative;
		font-family: PingFangSC-Regular, PingFang SC;
		overflow: auto;
		padding-bottom: 280px;

		.nav {
			padding: 40px 20px;

			.store {
				width: 100%;
				height: 250px;
				background: #ffffff;
				border-radius: 16px;
				display: flex;

				.storeImg {
					padding: 40px 0 0 30px;

					img {
						border-radius: 16px;
						width: 240px;
						height: 160px;
					}
				}

				.storedesc {
					width: 435px;
					margin: 15px 0;
					padding: 30px 10px 10px 30px;

					.name {
						font-size: 28px;
						text-overflow: ellipsis;
						overflow: hidden;
						white-space: nowrap;
						color: #333333;

						font-family: PingFangSC-Medium, PingFang SC;
						font-weight: 500;
					}

					.content {
						margin-top:8px;
						font-size: 22px;
						line-height: 35px;
						// text-overflow: ellipsis;
						// overflow: hidden;
						// white-space: nowrap;
						text-overflow: ellipsis;
						overflow: hidden;
						display: -webkit-box;
						-webkit-line-clamp: 2;
						-webkit-box-orient: vertical;
						color: #999999;
					}

					.price {
						padding-top: 15px;
						font-size: 32px;
						font-family: PingFangSC-Semibold, PingFang SC;
						font-weight: 600;

						.isusePrice {
							color: #FB7633;
						}

						.nousePrice {
							padding-left: 20px;
							font-size: 28px;
							color: #999999;
							text-decoration: line-through;
						}
					}
				}
			}

			.goods-diagnose {
				width: 100%;
				height: auto;
				background: #FFFFFF;
				border-radius: 16px;
				box-shadow: 0px 1px 10px 0px rgba(197, 197, 197, 0.29);
				display: flex;
				flex-direction: column;
				align-items: center;
				position: relative;
				padding-top: 8px;

				.goods-cell {
					width: calc(100% - 68px);
					height: auto;
					display: flex;
					flex-direction: column;

					&:last-child {
						border: none;
					}

					.line-up {
						margin-top: 38px;
						display: flex;
						align-items: center;

						&.show-limit {
							margin-top: 29px;
						}

						.goods-name {
							height: 28px;
							font-size: 26px;
							font-family: PingFang-SC-Bold, PingFang-SC;
							font-weight: bold;
							color: #333333;
							line-height: 28px;
							flex: 1;
						}

						.price-now {
							height: 32px;
							font-size: 26px;
							font-family: PingFangSC-Semibold, PingFang SC;
							font-weight: 600;
							color: #FB7633;
							line-height: 32px;
						}

						.price-old {
							height: 37px;
							margin-left: 10px;
							font-size: 26px;
							font-family: PingFangSC-Regular, PingFang SC;
							font-weight: 400;
							color: #999999;
							line-height: 37px;
							text-decoration: line-through;
						}
					}

					.line-down {
						display: flex;
						align-items: center;
						width: 100%;
						font-family: PingFangSC-Regular, PingFang SC;
						margin-top: 16px;

						span {
							height: 20px;
							font-size: 22px;
							font-weight: 400;
							color: #666666;
							line-height: 20px;

							&.time-left {
								color: #FB7633;
								margin-left: 8px;
							}
						}
					}

					.part-line {
						width: 100%;
						height: 1px;
						border-bottom: 1px solid #E6E6E6;
						margin-top: 30px;
					}
				}
			}

			.goodsList {
				margin: 20px 0;
				width: 100%;
				height: 340px;
				background: #ffffff;
				border-radius: 16px;
				padding: 34px;
				box-sizing: border-box;

				&.hideBalance {
					height: 280px;
				}

				.pricecolum {
					border-bottom: 1px dashed #dfdfdf;
					padding-bottom: 20px;

					font-size: 26px;
					font-family: PingFangSC-Regular, PingFang SC;
					font-weight: 400;
					color: #333333;

					.goodsPrice {
						width: 100%;
						height: 60px;
						line-height: 60px;

						font-size: 26px;
						color: #333333;

						.text {
							float: left;
						}

						.val {
							float: right;
							font-family: PingFangSC-Medium, PingFang SC;
							font-weight: 500;
						}
					}

					.goodsPoup {
						width: 100%;
						height: 60px;
						line-height: 60px;
						// padding-left: 20px;
						position: relative;

						.text {
							float: left;
						}

						.val {
							float: right;
							padding-right: 20px;
							color: #FB7633;

							font-size: 26px;
							font-family: PingFangSC-Medium, PingFang SC;
							font-weight: 500;
						}

						.valcolr {
							float: right;
							padding-right: 20px;
							color: #646464;
							font-size: 26px;

							font-family: PingFangSC-Medium, PingFang SC;
							font-weight: 500;
						}

						.imgpoup {
							position: absolute;
							right: 0;
							top: 22px;
							display: flex;

							img {
								height: 16px;
							}
						}

					}
				}

				.pricecount {
					padding: 35px 0 0 40px;

					.text {
						float: right;
						font-size: 26px;
						color: #333333;
						font-family: PingFangSC-Regular, PingFang SC;
						font-weight: 400;
						line-height: 37px;

						.val {
							color: #FB7633;
							font-size: 28px;

							font-family: PingFangSC-Medium, PingFang SC;
							font-weight: 500;
						}
					}
				}
			}

			.beanCake {
				width: 100%;
				height: 105px;
				background: #FFFFFF;
				border-radius: 16px;
				display: flex;
				align-items: center;
				justify-content: space-between;
				padding-left: 34px;
				padding-right: 34px;

				.beanCake-left {
					display: flex;
					align-items: center;

					img {
						width: 34px;
						height: 34px;
					}

					p {
						font-size: 26px;
						font-family: PingFangSC-Regular, PingFang SC;
						font-weight: 400;
						color: #333333;
						margin-left: 12px;
					}
				}

				.beanCake-right {
					display: flex;
					align-items: center;

					p {
						font-size: 26px;
						font-family: PingFangSC-Medium, PingFang SC;
						font-weight: 500;
						color: #333333;
						// margin-right: 12px;
					}

					.beanCake-radio {

						// width: 36px;
						// height: 36px;
					}
				}
			}
		}


		.foot {
			width: 100%;
			position: fixed;
			z-index: 100;
			bottom: 0;

			.foot-text {
				width: 100%;
				height: 55px;
				background: rgba(239, 234, 243, 1);
				color: #999999;
				font-size: 28px;
				position: relative;
				padding-left: 25px;
				font-size: 26px;
				font-family: PingFangSC-Regular, PingFang SC;
				font-weight: 400;
				display: flex;
				align-items: center;

				.buyText {
					color: #e76f51;
				}

				.textRadio {
					width: 36px;
					height: 36px;
					border: 1px solid #c8c9cc;
					border-radius: 50%;
					margin-right: 16px;

					.icon-checker {
						display: none;
					}

					&.clicked {
						border: none;
						background: #FB7633;
						position: relative;
						display: flex;
						justify-content: center;
						align-items: center;

						.icon-checker {
							display: block;
							width: 22px;
							margin-top: 2px;
						}

						.checker {
							width: 14px;
							height: 21px;
							border: 5px solid #ffffff;
							position: absolute;
							left: 50%;
							top: 50%;
							border-top: none;
							border-left: none;
							border-radius: 2px;
							transform: translate(-50%, -50%) rotate(40deg);
							margin-top: -2px;
							margin-left: -0.5px;
						}
					}
				}
			}

			.foot-btn {
				width: 100%;
				height: 200px;
				background: #f3f3f3;
				// min-height: 200px;

				.left {
					float: left;
					padding: 52px 0 0 54px;
					font-size: 30px;
					color: #333333;
					font-weight: 500;

					.payprice {
						color: #FB7633;

						font-size: 36px;
						font-family: PingFangSC-Semibold, PingFang SC;
						font-weight: 600;

						.bigger {
							font-size: 48px;

							font-family: PingFangSC-Semibold, PingFang SC;
							font-weight: 600;
						}

						.smaller {
							font-size: 36px;
						}
					}
				}

				.right {
					float: right;
					padding: 30px 56px 0 0;

					button {
						width: 260px;
						height: 88px;
						background: #FB7633;
						border-radius: 44px;
						border: none;
						font-size: 32px;
						color: #FFFFFF;
						font-weight: 500;
						font-family: PingFangSC-Medium, PingFang SC;
						font-weight: 500;
					}
				}
			}
		}

		.poup {
			width: 100%;
			position: relative;

			.van-action-sheet__content {
				background: #F5F5F5;
			}

			.sticky-head {
				position: sticky;
				top: 0;
				z-index: 10000;
				// background: #ffffff;

				padding-bottom: 20px;

				.header {
					width: 100%;
					text-align: center;
					padding: 24px 25px;
					position: relative;
					display: flex;
					align-items: center;
					background: #F5F5F5;

					.show {
						// float: left;
						// img{
						//     width: 126px;
						//     height: 33px;
						// }
						flex: 1;
						height: 37px;
						font-size: 26px;
						font-family: PingFangSC-Regular, PingFang SC;
						font-weight: 400;
						color: #999999;
						line-height: 37px;
						text-align: left;
						display: flex;
						align-items: center;

						img {
							width: 28px;
							height: 28px;
							margin-left: 12px;
						}
					}

					.title {
						img {
							width: 100px;
							height: 33px;
						}

						// padding-right: 60px;
						flex: 1;
						font-size: 34px;
						font-family: PingFangSC-Medium,
						PingFang SC;
						font-weight: 500;
						color: #000000;
						line-height: 24px;
					}

					.close {
						flex: 1;
						display: flex;
						justify-content: flex-end;

						img {
							width: 36px;
							height: 36px;
						}
					}
				}

				.tabChenge {
					width: 100%;
				}
			}

			.tabContent {
				width: 100%;

				.isUseContent {
					.poupcard {
						width: 100%;
						height: 400px;
						background-image: url('../../assets/douyuxingchen/storeList/poupImg.png');
						background-size: 100% 400px;

						.hedmain {
							font-size: 24px;
							width: 100%;
							//padding: 30px 0 0 62px;
							color: #333333;
							padding-left: 62px;

							.countNumber {
								.countnum {
									display: inline-block;
									padding: 40px 0 0 0;
								}

								.flright {
									float: right;
									padding: 30px 50px 0 0;

									.right {
										display: inline-block;
										width: 210px;
										height: 40px;
										background-image: url('../../assets/douyuxingchen/storeList/priImg@2x.png');
										background-size: 100% 100%;
										text-align: center;
										line-height: 40px;
									}
								}
							}
						}

						.cardhead {
							width: 100%;
							padding: 50px 0 0 62px;
							position: relative;

							.sumtext {
								color: #FB7633;
								font-size: 30px;


								.sum {
									font-size: 72px;
									font-family: PingFangSC-Medium, PingFang SC;
									font-weight: 500;
								}
							}

							.sumradio {
								position: absolute;
								right: 80px;
								top: 70px;
								
								.bxj-checker {
									width: 36px;
									height: 36px;
									border: 1px solid #c8c9cc;
									border-radius: 50%;
									margin-right: 16px;

									.icon-checker {
										display: none;
									}

									&.clicked {
										border: none;
										background: #FB7633;
										position: relative;
										display: flex;
										justify-content: center;
										align-items: center;

										.icon-checker {
											display: block;
											width: 22px;
											margin-top: 2px;
										}

										.checker {
											width: 14px;
											height: 21px;
											border: 5px solid #ffffff;
											position: absolute;
											left: 50%;
											top: 50%;
											border-top: none;
											border-left: none;
											border-radius: 2px;
											transform: translate(-50%, -50%) rotate(40deg);
											margin-top: -2px;
											margin-left: -0.5px;
										}
									}
								}
							}
						}

						.cardmain {
							font-size: 24px;
							width: 100%;
							padding: 50px 0 0 62px;
							color: #333333;

							.countdat {
								padding: 15px 0;

								.datcol {
									font-size: 22px;
									color: #B6B6B6;
								}

								.right {
									margin-right: 20px;
									display: inline-block;
									width: 120px;
									height: 40px;
									text-align: center;
									line-height: 40px;
									background: #FB7633;
									color: #FFFFFF;
								}
							}

							.countTip {
								padding-bottom: 10px;
								color: #333333;
								font-size: 24px;
							}
						}

						&.disable{
							.cardhead{
								.sumradio{
									.bxj-checker{
										background: #ebedf0;
									}
								}
							}
						}
					}
				}

				.noUseContent {
					.poupcard {
						width: 100%;
						height: 400px;
						background-image: url('../../assets/douyuxingchen/storeList/poupImg.png');
						background-size: 100% 400px;

						.hedmain {
							font-size: 24px;
							width: 100%;
							//padding: 30px 0 0 62px;
							color: #333333;
							padding-left: 62px;

							.countNumber {
								.countnum {
									display: inline-block;
									padding: 30px 0 0 0;
								}

								.flright {
									float: right;
									padding: 25px 50px 0 0;

									.right {
										display: inline-block;
										width: 210px;
										height: 40px;
										background: #E2E2E2;
										text-align: center;
										line-height: 40px;
										border-radius: 8px;
									}
								}
							}
						}

						.cardhead {
							width: 100%;
							padding: 50px 0 0 62px;
							position: relative;

							.sumtext {
								color: #C2C2C2;
								font-size: 30px;

								.sum {
									font-size: 72px;
								}
							}

							.sumradio {
								position: absolute;
								right: 65px;
								top: 40px;

								img {
									width: 110px;
								}
							}
						}

						.cardmain {
							font-size: 24px;
							width: 100%;
							padding: 80px 0 0 62px;
							color: #999999;

							.countdat {
								padding: 15px 0;
							}

							.countTip {
								padding-bottom: 10px;
								color: #B6B6B6;
							}
						}
					}
				}

				&.is_wallet {
					.isUseContent {
						display: flex;
						align-items: center;
						justify-content: center;

						.poupcard {
							width: 740px;
							height: 339px;
							background-image: url('~@/assets/wallet/wallet-panel-bg.svg');
							background-size: 100%;
							position: relative;

							.cardhead {
								.tiny-line {
									position: absolute;
									top: 174px;
									width: 640px;
									height: 2px;
									z-index: 100;
									left: 50px;
								}

								.sumradio {
									top: 80px;
								}
							}

							.desc {
								color: red;
								position: absolute;
								top: 206px;

								width: 632px;
								font-size: 24px;
								font-family: PingFangSC-Regular, PingFang SC;
								font-weight: 400;
								color: #999999;
								line-height: 40px;
								left: 50%;
								transform: translate(-50%, 0);
								text-align: justify;
							}
						}
					}
				}
			}

			.beanCake-action-sheet {
				height: 567px !important;

				.beanCake-sheet-top {
					font-size: 34px;
					font-family: PingFangSC-Medium, PingFang SC;
					font-weight: 500;
					color: #000000;
					text-align: center;
					margin-top: 36px;
				}

				.beanCake-sheet-toptitle {
					width: 590px;
					height: 92px;
					// border:1px solid red;
					font-size: 28px;
					font-family: PingFangSC-Regular, PingFang SC;
					font-weight: 400;
					color: #666666;
					line-height: 46px;
					margin: 0 auto;
					margin-top: 26px;
				}

				.beanCake-sheet-center {
					width: 600px;
					height: 80px;
					font-size: 24px;
					font-family: PingFangSC-Regular, PingFang SC;
					font-weight: 400;
					color: #a5a5a5;
					line-height: 40px;
					margin: 0 auto;
					// border:1px solid red;
					margin-top: 20px;

					.beanCake-sheet-center-img {
						width: 26px;
						height: 26px;
					}
				}

				.beanCake-sheet-btn {
					width: 632px;
					height: 88px;
					margin: 0 auto;
					margin-top: 40px;

					div {
						width: 632px;
						height: 88px;
						background: #FB7633;
						border-radius: 50px;
						text-align: center;
						line-height: 88px;
						font-family: PingFangSC-Regular, PingFang SC;
						font-weight: 400;
						font-size: 32px;
						color: #fff;
					}

					// img{
					//     width: 100%;
					//     height: 100%;
					// }
				}

				.beanCake-sheet-close {
					width: 148px;
					height: 45px;
					// border:1px solid black;
					font-size: 32px;
					font-family: PingFangSC-Medium, PingFang SC;
					font-weight: 500;
					color: #999999;
					line-height: 45px;
					margin: 0 auto;
					text-align: center;
					margin-top: 20px;
				}
			}

			.sticky-foot {
				position: sticky;
				bottom: 0;
				z-index: 10000;
				// background: #ffffff;
				width: 100%;
				height: 150px;

				.confirmBtn {
					width: 100%;
					text-align: center;
					padding: 20px 0;

					button {
						width: 672px;
						height: 88px;
						background: #FB7633;
						border-radius: 40px;
						border: none;
						font-size: 36px;
						color: #FFFFFF;
						font-weight: 500;
					}
				}
			}

			.beanCake-foot {
				position: absolute;
				bottom: 0 !important;
			}
		}

		.main-loading {
			width: 100%;
			height: 100%;
			display: flex;
			justify-content: center;
			align-items: center;
			position: fixed;
			z-index: 1000;
			top: 0;
			left: 0;
		}
	}
</style>
<style>
	.van-tabs__nav {
		background: #F5F5F5 !important;
	}

	.van-popup {
		height: auto !important;
		max-height: 60% !important;
		padding-bottom: 30px !important;
	}
</style>