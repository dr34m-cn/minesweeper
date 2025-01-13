<template>
	<div class="main" style="background-color: black;color: #FFFFFF;">
		<div class="main-top">
			<el-button type="success" @click="setSize(9, 9, 10)">基础</el-button>
			<el-button type="warning" @click="setSize(16, 16, 40)">中级</el-button>
			<el-button type="danger" @click="setSize(30, 16, 99)">专家</el-button>
			<!-- <el-button type="info">自定义</el-button>
			<el-button type="danger" @click="setSize(16, 30, 99)">专家（纵向）</el-button> -->
		</div>
		<div class="main-center">
			<div class="main-content-box">
				<div class="m">
					<div class="m-top">
						<div class="m-top-num">
							<el-image :src="imgFontNum[Math.floor(num / 1000000) % 10]" v-if="num > 999999" />
							<el-image :src="imgFontNum[Math.floor(num / 100000) % 10]" v-if="num > 99999" />
							<el-image :src="imgFontNum[Math.floor(num / 10000) % 10]" v-if="num > 9999" />
							<el-image :src="imgFontNum[Math.floor(num / 1000) % 10]" v-if="num > 999" />
							<el-image :src="imgFontNum[Math.floor(num / 100) % 10]" />
							<el-image :src="imgFontNum[Math.floor(num / 10) % 10]" />
							<el-image :src="imgFontNum[num % 10]" />
						</div>
						<div class="imgbox">
							<div class="eImg" @click="setDefault">
								<el-image style="width: 21px; height: 21px;" :src="imgSmile[status]"></el-image>
							</div>
						</div>
						<div class="m-top-num">
							<el-image :src="imgFontNum[Math.floor(time / 1000000) % 10]" v-if="time > 999999" />
							<el-image :src="imgFontNum[Math.floor(time / 100000) % 10]" v-if="time > 99999" />
							<el-image :src="imgFontNum[Math.floor(time / 10000) % 10]" v-if="time > 9999" />
							<el-image :src="imgFontNum[Math.floor(time / 1000) % 10]" v-if="time > 999" />
							<el-image :src="imgFontNum[Math.floor(time / 100) % 10]" />
							<el-image :src="imgFontNum[Math.floor(time / 10) % 10]" />
							<el-image :src="imgFontNum[time % 10]" />
						</div>
					</div>
					<div class="m-content">
						<div class="m-content-line" v-for="line in boxList">
							<div class="m-content-line-item" v-for="item in line" @contextmenu="rightClick($event, item)"
								@click="clickItem(item)">
								<template v-if="item.open">
									<!-- 打开的 -->
									<template v-if="item.isBoom">
										<!-- 是炸弹 -->
										<el-image :src="imgBoxType[2]" v-if="item.boomClick"></el-image>
										<el-image :src="imgBoxType[3]" v-else></el-image>
									</template>
									<el-image :src="imgBoomNum[item.num]" v-else></el-image>
								</template>
								<template v-else>
									<!-- 没打开 -->
									<el-image :src="imgBoxType[1]" v-if="item.flag"></el-image>
									<el-image :src="imgBoxType[0]" v-else></el-image>
								</template>
							</div>
						</div>
					</div>
				</div>
			</div>
			<div class="main-bottom">
				<span style="margin-right: 5px;">自动标雷</span>
				<el-switch v-model="autoSetBoom">
				</el-switch>
				<el-button type="success" style="margin-left: 20px;" @click="autoOpen">一键翻开</el-button>
			</div>
		</div>

	</div>
</template>

<script>
	import {
		imgSmile,
		imgFontNum,
		imgBoomNum,
		imgBoxType
	} from '@/utils/img.js';
	export default {
		components: {},
		data() {
			return {
				time: 0, // 时间
				autoSetBoom: true, // 自动标雷
				timer: null, // 计时器
				x: 16, // 有多少列
				y: 16, // 有多少行
				defaultNum: 40, // 规则雷数
				num: 40, // 剩余雷的数量
				status: 0, // 0-正常，1-踩雷，2-成功
				imgSmile, // 0-笑脸，1-失败，2-成功
				imgFontNum, // 机械数字0-9
				imgBoomNum, // 周围雷数0-8
				startFlag: false, // 开始标志
				imgBoxType, // 0-正常，1-标旗子，2-点爆的雷，3-雷
				boxList: [] // 格子列表
			};
		},
		created() {
			this.setDefault();
		},
		watch: {
			boxList: {
				handler(newVal, oldVal) {
					this.checkSuccess(newVal);
					this.calcLastBoom(newVal);
				},
				deep: true
			}
		},
		methods: {
			setDefault() {
				this.setSize(this.x, this.y, this.defaultNum);
			},
			setSize(x, y, num) {
				this.stopTimeCount();
				this.startFlag = false;
				this.status = 0;
				this.time = 0;
				this.x = x;
				this.y = y;
				this.defaultNum = num;
				this.num = num;
				let boxList = [];
				for (let yi = 0; yi < y; yi++) {
					let xList = [];
					for (let xi = 0; xi < x; xi++) {
						xList.push({
							'x': xi, // 左起横坐标（从0开始）
							'y': yi, // 上起纵坐标（从0开始）
							open: false, // 是否打开
							clear: false, // 周围已清空
							flag: false, // 是否标旗
							isBoom: false, // 是否是雷
							num: 0, // 周围雷数
							highlight: false, // 是否高亮显示
							boomClick: false // 是否是点爆的雷
						})
					}
					boxList.push(xList);
					this.boxList = boxList;
				}
			},
			// 右击某个格子
			rightClick(event, item) {
				event.preventDefault();
				item.flag = !item.flag;
			},
			// 检查标记成功
			checkSuccess(box) {
				let unOpenSize = 0;
				for (let y = 0; y < this.y; y++) {
					let line = box[y].filter(item => !item.open);
					if (line && line.length > 0) {
						unOpenSize += line.length;
					}
				}
				if (unOpenSize == this.defaultNum) {
					this.status = 2;
					this.stopTimeCount();
				}
			},
			// 计算剩余雷数
			calcLastBoom(box) {
				let flags = 0;
				for (let y = 0; y < this.y; y++) {
					let line = box[y].filter(item => item.flag);
					if (line && line.length > 0) {
						flags += line.length;
					}
				}
				let num = this.defaultNum - flags;
				if (num <= 0) {
					this.num = 0;
				} else {
					this.num = num;
				}
			},
			// 点击某个格子,返回999表示结束
			clickItem(item, isAuto = false) {
				// isAuto表示是否自动点击的
				if (this.startFlag) {
					if (this.status != 0) {
						// 游戏已经结束
						return 999;
					}
					if (item.open) {
						if (!isAuto) {
							this.openOrSee(item);
							// TODO 手动点已经翻开的格子
						}
						return
					}
					if (item.flag) {
						if (!isAuto) {
							this.boxList[item.y][item.x].flag = false;
							this.$forceUpdate();
						}
						return
					}
					this.boxList[item.y][item.x].open = true;
					this.$forceUpdate();
					if (item.isBoom) {
						this.boxList[item.y][item.x].boomClick = true;
						this.status = 1;
						this.stopTimeCount();
						this.showAllBoom();
						return 999;
					} else {
						let around = this.findAround(item);
						if (item.num == 0) {
							// 点开数字是0
							for (let i = 0; i < around.length; i++) {
								let rst = this.clickItem(around[i], true);
								if (rst == 999) {
									break;
								}
							}
						} else {
							if (this.autoSetBoom) {
								this.autoBoom();
							}
						}
					}
				} else {
					// 首次点击
					this.clickFirst(item);
				}
			},
			openOrSee(item, needSee = true) {
				// 翻开或查看周围
				// 操作标识
				let flag = false;
				if (item.num == 0 || item.clear) {
					return
				}
				let around = this.findAround(item);
				let flags = around.filter(each => each.flag);
				if (flags && item.num == flags.length) {
					let unflags = around.filter(each => !each.flag && !each.open);
					if (unflags) {
						flag = true;
						for (let i = 0; i < unflags.length; i++) {
							let rst = this.clickItem(unflags[i], true);
							if (rst == 999) {
								break;
							}
						}
						flag = true;
					}
				}
				if (flag) {
					this.boxList[item.y][item.x].clear = true;
				} else if (needSee) {
					// TODO 查看周围
				}
				this.$forceUpdate();
				return flag;
			},
			autoOpen() {
				let flag = false;
				for (let x = 0; x < this.x; x++) {
					for (let y = 0; y < this.y; y++) {
						let item = this.boxList[y][x];
						if (item.open && !item.clear) {
							let fg = this.openOrSee(item, false);
							if (fg) {
								flag = true;
							}
						}
					}
				}
				if (flag) {
					this.autoOpen();
				}
			},
			autoBoom() {
				// 自动标雷
				for (let x = 0; x < this.x; x++) {
					for (let y = 0; y < this.y; y++) {
						let item = this.boxList[y][x];
						if (item.open && item.num != 0) {
							let around = this.findAround(item);
							let unOpen = around.filter(each => each.open == false);
							if (unOpen.length == item.num) {
								for (let i = 0; i < unOpen.length; i++) {
									this.boxList[unOpen[i].y][unOpen[i].x].flag = true;
								}
								this.$forceUpdate();
							}
						}
					}
				}
			},
			// 展示所有雷
			showAllBoom() {
				for (let x = 0; x < this.x; x++) {
					for (let y = 0; y < this.y; y++) {
						// 是雷并且没有打开并且没有标记
						if (this.boxList[y][x].isBoom && !this.boxList[y][x].open && !this.boxList[y][x].flag) {
							this.boxList[y][x].open = true;
						}
						// 不是雷但标记了
						if (this.boxList[y][x].flag && !this.boxList[y][x].isBoom) {
							// TODO 后续处理
						}
					}
				}
				this.$forceUpdate();
			},
			// 首次点击，确保点击的附近没有雷，随机放置雷，开始计时，修改标志
			clickFirst(item) {
				// 开始计时
				this.startTimeCount();
				// 修改标志
				this.startFlag = true;
				// 随机放雷
				// 无雷的格子
				let safe = this.findAround(item, true);
				// 雷的序号
				const nums = this.getRandomNums(this.x * this.y, this.defaultNum);
				let i = 0;
				for (let y = 0; y < this.y; y++) {
					for (let x = 0; x < this.x; x++) {
						let isSafe = safe.find(item => item.x == x && item.y == y);
						if (isSafe) {
							continue;
						}
						if (nums.includes(i)) {
							this.boxList[y][x].isBoom = true;
						}
						i++;
					}
				}
				// 计算周围雷数
				for (let y = 0; y < this.y; y++) {
					for (let x = 0; x < this.x; x++) {
						this.boxList[y][x].num = this.countAround(x, y);
					}
				}
				this.$forceUpdate();
				this.clickItem(item);
			},
			// 随机放雷
			getRandomNums(all, count) {
				// all-总数，count-雷数
				const nums = new Set();
				while (nums.size < count) {
					const num = Math.floor(Math.random() * (all - 9));
					nums.add(num);
				}
				return Array.from(nums);
			},
			// 计算周围雷数
			countAround(x, y) {
				let around = this.findAround({
					x,
					y
				});
				let count = 0;
				for (let i = 0; i < around.length; i++) {
					if (around[i].isBoom) {
						count++;
					}
				}
				return count;
			},
			// 找到周围坐标
			findAround(item, includeSelf = false) {
				const x = item.x;
				const y = item.y;
				let rst = [];
				for (let i = -1; i < 2; i++) {
					let cux = x + i;
					if (cux < 0 || cux >= this.x) {
						continue;
					}
					for (let j = -1; j < 2; j++) {
						if (i == 0 && j == 0 && !includeSelf) {
							continue;
						}
						let cuy = y + j;
						if (cuy < 0 || cuy >= this.y) {
							continue;
						}
						rst.push(this.boxList[cuy][cux]);
					}
				}
				return rst;
			},
			// 开始计时
			startTimeCount() {
				this.timer = setInterval(() => {
					this.time = this.time + 1;
				}, 1000);
			},
			// 停止计时
			stopTimeCount() {
				if (this.timer != null) {
					clearInterval(this.timer);
					this.timer = null;
				}
			}
		}
	};
</script>
<style scoped lang="scss">
	.main {
		width: 100%;
		height: 100%;
		box-sizing: border-box;
		overflow-y: auto;

		.main-top {
			height: 80px;
			box-sizing: border-box;
			display: flex;
			align-items: center;
			justify-content: center;
		}

		.main-center {
			height: calc(100% - 80px);
			box-sizing: border-box;
			overflow-y: auto;

			.main-content-box {
				display: flex;
				justify-content: center;

				.m {
					padding: 9px;
					border-radius: 6px;
					background: #cccccc;

					.m-top {
						display: flex;
						justify-content: space-between;
						align-items: center;
						border: 2px solid #808080;
						border-right-color: #ffffff;
						border-bottom-color: #ffffff;
						padding: 2px 1px;
						position: relative;

						.m-top-num {
							display: flex;
							align-items: center;
							background-color: #000000;

							:deep(.el-image) {
								width: 13px;
								height: 23px;
							}
						}

						.imgbox {
							position: absolute;
							width: 100%;
							height: 100%;
							align-items: center;
							display: flex;
							justify-content: center;
							z-index: 9;

							.eImg {
								width: 25px;
								height: 25px;
								border: 2px solid #ffffff;
								border-right-color: #808080;
								border-bottom-color: #808080;
								box-sizing: border-box;
							}

							.eImg:active {
								transform: translate(1px, 1px);
								border: 2px solid #808080;
								border-right: 1px solid #808080;
								border-bottom: 1px solid #808080;
							}
						}
					}

					.m-content {
						margin-top: 9px;
						border: 2px solid #808080;
						border-right-color: #ffffff;
						border-bottom-color: #ffffff;

						.m-content-line {
							display: flex;

							.m-content-line-item {
								width: 25px;
								height: 25px;
							}
						}
					}
				}
			}

			.main-bottom {
				height: 80px;
				display: flex;
				align-items: center;
				justify-content: center;
			}
		}
	}
</style>