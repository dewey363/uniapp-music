<template>
	<view class="search">
		<!-- 头部组件 -->
		<musichead title="搜索" :icon="true" :iconblack="true"></musichead>
		<!-- 通用容器 -->
		<view class="container">
			<!-- 滚动区域 -->
			<scroll-view scroll-y="true">
				<!-- 搜索 -->
				<view class="search-search">
					<!-- 搜索图标 -->
					<text class="iconfont iconsearch"></text>
					<!-- 搜索表单  双向数据绑定-->
					<input type="text" placeholder="搜索歌曲" v-model="searchWord" @confirm="handleToSearch(searchWord)"
						@input="handleToSuggest">
					<!-- 搜索关闭图标 -->
					<text v-show="searchType!=1" class="iconfont iconguanbi" @tap="handleToClose"></text>
				</view>

				<!-- 块占位符  -->
				<block v-if="searchType==1">
					<!-- 历史记录 -->
					<view class="search-history">
						<!-- 头部：左右 -->
						<view class="search-history-head">
							<text>历史记录</text>
							<!-- 垃圾桶图标：清空历史记录 -->
							<text class="iconfont iconlajitong" @tap="handleToClear"></text>
						</view>
						<!-- 列表 历史记录 -->
						<view class="search-history-list">
							<view v-for="(item,index) in searchHistory" :key="index" @tap="handleToWord(item)">{{item}}
							</view>
						</view>
					</view>

					<!-- 热搜榜 -->
					<view class="search-hot">
						<!-- 头部 -->
						<view class="search-hot-head">热搜榜</view>
						<!-- 列表项：左中右 -->
						<view class="search-hot-item" v-for="(item,index) in searchHot" :key="index"
							@tap="handleToWord(item.searchWord)">
							<view class="search-hot-top">{{index+1}}</view>
							<view class="search-hot-word">
								<view>
									{{item.searchWord}}
									<image :src="item.iconUrl" mode="aspectFill"></image>
								</view>
								<view>{{item.content}}</view>
							</view>
							<text class="search-hot-count">{{item.score}}</text>
						</view>
					</view>
				</block>

				<!-- 搜索结果 -->
				<block v-else-if="searchType==2">
					<view class="search-result">
						<!-- 搜索结果项 -->
						<view class="search-result-item" v-for="(item,index) in searchList" :key="index"
							@tap="handleToDetail(item.id)">
							<!-- 搜索结果词 -->
							<view class="search-result-word">
								<view>{{item.name}}</view>
								<view>{{item.artists[0].name}} - {{item.album.name}}</view>
							</view>
							<!-- 搜索结果播放图标 -->
							<text class="iconfont iconbofang"></text>
						</view>
					</view>
				</block>

				<!-- 搜索下拉提示 -->
				<block v-else-if="searchType==3">
					<view class="search-suggest">
						<!-- 头部 -->
						<view class="search-suggest-head">搜索”{{searchWord}}“</view>
						<!-- 列表项 搜索提示 -->
						<view class="search-suggest-item" v-for="(item,index) in searchSuggest" :key="index"
							@tap="handleToWord(item.keyword)">
							<text class="iconfont iconsearch"></text> {{item.keyword}}
						</view>

					</view>
				</block>
			</scroll-view>
		</view>
	</view>
</template>

<script>
	// 字体图标引入
	import '@/common/iconfont.css'
	// 头部组件引入
	import musichead from '../../components/musichead/musichead.vue'
	// 搜索页相关接口引入
	import {
		searchHot,
		searchWord,
		searchSuggest
	} from '../../common/api.js'

	export default {
		data() {
			return {
				// 搜索榜
				searchHot: [],
				// 搜索词
				searchWord: '',
				// 历史记录
				searchHistory: [],
				// 搜索类型 1是默认显示,2是显示搜索结果,3是显示搜索提示
				searchType: 1,
				// 搜索结果
				searchList: [],
				// 搜索下拉提示
				searchSuggest: []
			}
		},
		components: {
			musichead
		},

		onLoad() {
			// 搜索榜数据
			searchHot().then(res => {
				// console.log(res[1])
				if (res[1].data.code == '200') {
					this.searchHot = res[1].data.data
				}
			})
		},

		methods: {
			// 点击热搜榜，搜索框就更新热搜词
			handleToWord(word) {
				this.searchWord = word
				this.handleToSearch(word)
			},
			// 点击搜索框，将输入结果存入搜索历史记录数组
			handleToSearch(word) {
				// console.log(word)
				this.searchHistory.unshift(word)
				// 相同记录进行数组元素去重
				this.searchHistory = [...new Set(this.searchHistory)]
				// 限制数组存入数量
				if (this.searchHistory.length > 10) {
					this.searchHistory.length = 10
				}

				// 本地存储 历史记录存储
				uni.setStorage({
					key: 'searchHistory',
					data: this.searchHistory
				})

				// 本地存储 历史记录取出
				uni.getStorage({
					key: 'searchHistory',
					success: res => {
						this.searchHistory = res.data
					}
				})

				// 搜索结果触发
				this.getSearchList(word)
			},

			// 点击垃圾桶 清空历史记录 清除本地存储
			handleToClear() {
				uni.clearStorage({
					key: 'searchHistory',
					success: res => {
						// 清空数组
						this.searchHistory = []
					}
				})
			},

			// 获取搜索结果
			getSearchList(word) {
				searchWord(word).then(res => {
					// console.log(res)
					if (res[1].data.code == '200') {
						this.searchList = res[1].data.result.songs
						this.searchType = 2
					}
				})
			},

			// 点击关闭按钮: 清空搜索词并返回搜索类型1
			handleToClose() {
				this.searchWord = '',
					this.searchType = 1
			},

			// 点击搜索结果项，跳转到详情页
			handleToDetail(songId) {
				uni.navigateTo({
					url: '/pages/detail/detail?songId=' + songId
				})
			},

			// 点击输入框，触发搜索下拉提示
			handleToSuggest(ev) {
				let value = ev.detail.value
				// console.log(value)
				if (!value) {
					this.searchType = 1;
					return;
				}
				// 发起搜索下拉提示请求
				searchSuggest(value).then(res => {
					if (res[1].data.code == '200') {
						this.searchSuggest = res[1].data.result.allMatch
						this.searchType = 3
					}
				})
			},

		}
	}
</script>

<style>
	/* 搜索 */
	.search-search {
		display: flex;
		align-items: center;
		height: 70rpx;
		margin: 70rpx 30rpx 50rpx 30rpx;
		background: #F7F7F7;
		border-radius: 50rpx;
	}

	/* 搜索 图标 */
	.search-search text {
		margin: 0 26rpx;
		color: #000000;
	}

	/* 搜索 表单输入框 */
	.search-search input {
		flex: 1;
		font-size: 26rpx;
		color: #000000;
	}

	/* 历史记录 */
	.search-history {
		margin: 0rpx 30rpx 50rpx 30rpx;
		font-size: 26rpx;
	}

	/* 历史记录 头部 */
	.search-history-head {
		display: flex;
		justify-content: space-between;
		margin-bottom: 36rpx;
	}

	/* 历史记录 列表 */
	.search-history-list {
		display: flex;
		flex-wrap: wrap;
	}

	.search-history-list view {
		padding: 16rpx 28rpx;
		border-radius: 40rpx;
		margin-right: 30rpx;
		margin-bottom: 30rpx;
		background: #f7f7f7;
	}

	/* 热搜榜 */
	.search-hot {
		margin: 0 30rpx;
		font-size: 26rpx;
	}

	/* 热搜榜 头部 */
	.search-hot-head {
		margin-bottom: 36rpx;
	}

	/* 热搜榜 列表项 */
	.search-hot-item {
		display: flex;
		align-items: center;
		margin-bottom: 58rpx;
	}

	/* 左 */
	.search-hot-top {
		color: #fb2222;
		width: 60rpx;
		margin-left: 8rpx;
	}

	/* 中 */
	.search-hot-word {
		flex: 1;
	}

	.search-hot-word view:nth-child(1) {
		font-size: 30rpx;
		color: #000000;
	}

	.search-hot-word view:nth-child(2) {
		font-size: 24rpx;
		color: #878787;
		/* 单行溢出隐藏 */
		width: 60vw;
		white-space: nowrap;
		overflow: hidden;
		text-overflow: ellipsis;
	}

	.search-hot-word image {
		width: 48rpx;
		height: 22rpx;
	}

	/* 右 */
	.search-hot-count {
		color: #878787;
	}

	/* 搜索结果 */
	.search-result {
		border-top: 2rpx solid #e4e4e4;
		padding: 30rpx;
	}

	.search-result-item {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding-bottom: 30rpx;
		margin-bottom: 30rpx;
		border-bottom: 2rpx solid #e4e4e4;
	}

	.search-result-word {}

	.search-result-word view:nth-child(1) {
		font-size: 28rpx;
		color: #235790;
		margin-bottom: 12rpx;
	}

	.search-result-word view:nth-child(2) {
		font-size: 22rpx;
		color: #898989;
	}

	.search-result-item text {
		font-size: 50rpx;
	}


	/* 搜索下拉提示 */
	.search-suggest {
		border-top: 2rpx solid #E4E4E4;
		padding: 30rpx;
		font-size: 26rpx;

	}

	.search-suggest-head {
		color: #4574a5;
		margin-bottom: 74rpx;
	}

	.search-suggest-item {
		color: #5d5d5d;
		margin-top: 74rpx;
	}

	.search-suggest-item text {
		color: #bdbdbd;
		margin-right: 28rpx;
		position: relative;
		top: 2rpx;
	}
</style>
