<template>
	<div class="main" style="background-color: black;">
		<div class="main-top">
			<el-button type="success" @click="setSize(9, 9, 10)">基础</el-button>
			<el-button type="warning" @click="setSize(16, 16, 40)">中级</el-button>
			<el-button type="danger" @click="setSize(30, 16, 99)">专家</el-button>
			<el-button type="info">自定义</el-button>
			<el-button type="danger" @click="setSize(16, 30, 99)">专家（纵向）</el-button>
		</div>
		<div class="main-center">
			<div class="main-content-box">
				<div class="m">
					<div class="m-top">
						<div class="imgbox">
							<el-image style="width: 21px; height: 21px;" :src="img[status]"></el-image>
						</div>
					</div>
					<div class="m-content">
						<div class="m-content-line" v-for="line in boxList">
							<div class="m-content-line-item" v-for="item in line">
							</div>
						</div>
					</div>
				</div>
			</div>
			<div class="main-bottom">
				<el-button type="primary">一键标雷</el-button>
				<el-button type="success">一键翻开</el-button>
			</div>
		</div>

	</div>
</template>

<script>
	export default {
		components: {},
		data() {
			return {
				x: 9,
				y: 9,
				num: 10,
				status: 0, // 0-正常，1-踩雷，2-成功
				img: [ // 0-笑脸，1-失败，2-成功
					'data:img/gif;base64,R0lGODlhFQAVAJEAAAAAAP//AL29vQAAACH5BAAHAP8ALAAAAAAVABUAAAJAlI+py50AoUMwWCsduBy33XXAAoaiUlZY+nBq8MKUSY9HSbtzft4X/vu1MCLhcBXRoXgyBlD5AWYmgsiUis0yCgA7',
					'data:img/gif;base64,R0lGODlhFQAVAJEAAAAAAP//AMDAwAAAACH5BAAHAP8ALAAAAAAVABUAAAJDlI+py50AoUMwWCsduBy33XXAAm5gYHrPdWLs6p6parShSN36eNg6LbBFKhxejyV6wXwo4PGH+vyMDCKLqhlOttxuAQA7',
					'data:img/gif;base64,R0lGODlhFQAVAJEAAAAAAP//AMDAwICAACH5BAAHAP8ALAAAAAAVABUAAAJHlI+py50AoUMwWCsduBy33XXAAoaiAkZqFYwUprZx+27xTR/swMoD/DKZXIaSkEMsXoytJQo58zyPOdIxSWJmJoIV9wueFAAAOw=='
				],
				boxList: []
			};
		},
		created() {
			this.setSize(this.x, this.y, this.num);
		},
		methods: {
			setSize(x, y, num) {
				let boxList = [];
				for (let yi = 0; yi < y; yi++) {
					let xList = [];
					for (let xi = 0; xi < x; xi++) {
						xList.push({
							'x': xi, // 左起横坐标（从0开始）
							'y': yi, // 上起纵坐标（从0开始）
							open: false, // 是否打开
							flag: false, // 是否标旗
							isBoom: false, // 是否是雷
							num: 0 // 周围雷数
						})
					}
					boxList.push(xList);
					this.boxList = boxList;
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

						.imgbox {
							width: 25px;
							height: 25px;
							border: 2px solid #ffffff;
							border-right-color: #808080;
							border-bottom-color: #808080;
							box-sizing: border-box;
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
								box-sizing: border-box;
								border: 3px solid #ffffff;
								border-right-color: #808080;
								border-bottom-color: #808080;
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