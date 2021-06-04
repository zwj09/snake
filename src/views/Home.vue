<template>
	<div class="home">
		<div>贪吃蛇</div>
		<div>
			当前分数是：<span v-text="score"></span>
		</div>
		<canvas ref="game" id="c_wh"> </canvas>
		<!-- 开始游戏按钮 -->
		<div>
			<button v-if="!starting" @click="startGame">开始游戏</button>
			<button v-else @click="pauseGame">暂停游戏</button>
			<button @click="reStart">重新开始</button>
		</div>
		<br />
		<div>
			<div class="center">
				<div class="gameButton" @click="buttonChangeDirection('↑')">↑</div>
			</div>
			<div class="center">
				<div class="gameButton" @click="buttonChangeDirection('←')">←</div>
				<div class="mid"></div>
				<div class="gameButton" @click="buttonChangeDirection('→')">→</div>
			</div>
			<div class="center">
				<div class="gameButton" @click="buttonChangeDirection('↓')">↓</div>
			</div>
		</div>
	</div>
</template>

<style scoped>
	.center {
		display: flex;
		justify-content: center;
	}

	.gameButton {
		cursor: pointer;
		width: 40px;
		height: 40px;
		background-color: lightskyblue;
		border-radius: 20%;
		align-content: center;
		box-shadow: 3px 3px 10px 5px rgb(128, 128, 128);
	}

	.gameButton:hover {
		background-color: rgba(243, 236, 236, 0.938);
	}

	.gameButton:active {
		background-color: rgba(112, 109, 109, 0.938);
	}

	.mid {
		width: 40px;
	}
</style>

<script>
	export default {
		
		mounted() {
			//监听键盘事件
			document.addEventListener("keydown", this.changeDirection);
			const CANVAS_BORDER_COLOR = 'black';
			const CANVAS_BACKGROUND_COLOR = 'white';
			console.log(this.$refs.game);
			this.game = this.$refs.game;
			this.context = this.game.getContext('2d');
			//画布
			const c = document.getElementById("c_wh");
			c.width = this.C_Width() * 400;
			c.height = this.C_Height() * 400;
			console.log(c);
			this.context.fillStyle = CANVAS_BACKGROUND_COLOR;
			this.context.strokeStyle = CANVAS_BORDER_COLOR;
			this.context.fillRect(0, 0, this.game.width, this.game.height);
			this.context.strokeRect(0, 0, this.game.width, this.game.height);
			this.drawSnake();
			this.addFood();
			console.log(this.screenWidth);
			console.log(this.C_Width());
			
		
		},
		data() {
			return {
				snake: [{
						x: 150,
						y: 150
					},
					{
						x: 140,
						y: 150
					},
					{
						x: 130,
						y: 150
					},
					{
						x: 120,
						y: 150
					},
					{
						x: 110,
						y: 150
					},
				],
				game: {},
				context: {},
				speedX: 10,
				speedY: 0,
				food: {
					x: 0,
					y: 0
				},
				foodArr: [],
				timer: {},
				starting: false,
				score: -1,
				screenWidth: window.innerWidth,
				// bili: this.screenWidth ,
				c_width: {},
			}
		},
		methods: {
			C_Width() {
				return this.screenWidth / 500.00
			},
			C_Height() {
				return this.screenWidth / 500.00
			},
			//开始游戏
			startGame() {
				this.starting = true;
				this.timer = setInterval(() => {
					this.clearCanvas();
					this.drawFood();
					this.advanceSnake();
					this.drawSnake();
					this.drawFoodArr();
				}, 100)
			},
			//暂停游戏
			pauseGame() {
				this.starting = false;
				clearInterval(this.timer);
			},
			//重新开始
			reStart() {
				location.reload();
			},
			//蛇身体每个部分
			drawSnakePart(snakePart) {
				this.context.fillStyle = 'lightgreen';
				this.context.strokeStyle = 'darkgreen';
				this.context.fillRect(snakePart.x, snakePart.y, 10, 10);
				this.context.strokeRect(snakePart.x, snakePart.y, 10, 10);
			},
			//画整条蛇
			drawSnake() {
				this.snake.forEach(this.drawSnakePart);
			},
			//让蛇动
			advanceSnake() {
				//新建一个头部，添加到this.snake中
				let ifPop = true;
				let head = {
					x: this.snake[0].x + this.speedX,
					y: this.snake[0].y + this.speedY,
				}
				this.snake.unshift(head);
				//通过判断是否吃到食物，选择是否取消尾部
				let didEatFood = (this.snake[0].x === this.food.x) &&
					(this.snake[0].y === this.food.y);
				this.foodArr.find((food, index) => {
					if (head.x === food.x && head.y === food.y) {
						this.foodArr.splice(index, 1);
						ifPop = false
					}
					return head.x === food.x && head.y === food.y;
				})
				if (didEatFood) {
					//让食物随机一个位置
					this.addFood();
				} else {
					if (ifPop) this.snake.pop();
				}
				this.snake.find((body, index) => {
					if ((head.x === body.x && head.y === body.y) && index > 2) {
						this.foodArr = this.foodArr.concat(this.snake.slice(index));
						this.snake = this.snake.slice(0, index);

					}
					return (head.x === body.x && head.y === body.y) && index > 2
				})
			},
			//更改食物的位置
			addFood() {
				//分数增加
				this.score += 1;
				//随机生成位置
				this.food.x = this.randomTen(0, this.game.width - 10);
				this.food.y = this.randomTen(0, this.game.height - 10);
				//判断生成的位置是否在蛇身上，如果在就重新生成
				let ifSame = this.snake.find(body => body.x === this.food.x && body.y === this.food.y);
				let ifFoodArr = this.foodArr.find(body => body.x === this.food.x && body.y === this.food.y);
				if (ifSame || ifFoodArr) this.addFood();
			},
			randomTen(max, min) {
				return Math.round((Math.random() * (max - min) + min) / 10) * 10;
			},
			//画蛇尸体
			drawFoodArr() {
				this.foodArr.forEach(food => {
					this.context.fillStyle = 'aqua';
					this.context.strokeStyle = 'darkred';
					this.context.fillRect(food.x, food.y, 10, 10);
					this.context.strokeRect(food.x, food.y, 10, 10);
				})
			},
			//画食物
			drawFood() {
				this.context.fillStyle = 'pink';
				this.context.strokeStyle = 'darkred';
				this.context.fillRect(this.food.x, this.food.y, 10, 10);
				this.context.strokeRect(this.food.x, this.food.y, 10, 10);
			},
			//清空画布
			clearCanvas() {
				this.context.fillStyle = 'white';
				this.context.strokeStyle = 'black';
				this.context.fillRect(0, 0, this.game.width, this.game.height);
				this.context.strokeRect(0, 0, this.game.width, this.game.height);
			},
			//键盘事件
			changeDirection(event) {
				const LEFT_KEY = 37;
				const RIGHT_KEY = 39;
				const UP_KEY = 38;
				const DOWN_KEY = 40;
				const keyPressed = event.keyCode;
				const goingUp = this.speedY === -10;
				const goingDown = this.speedY === 10;
				const goingRight = this.speedX === 10;
				const goingLeft = this.speedX === -10;
				if (keyPressed === LEFT_KEY && !goingRight) {
					this.speedX = -10;
					this.speedY = 0;
				}
				if (keyPressed === UP_KEY && !goingDown) {
					this.speedX = 0;
					this.speedY = -10;
				}
				if (keyPressed === RIGHT_KEY && !goingLeft) {
					this.speedX = 10;
					this.speedY = 0;
				}
				if (keyPressed === DOWN_KEY && !goingUp) {
					this.speedX = 0;
					this.speedY = 10;
				}
			},
			//按钮事件
			buttonChangeDirection(direction) {
				const LEFT_STRING = "←";
				const UP_STRING = "↑";
				const RIGHT_STRING = "→";
				const DOWN_STRING = "↓";
				const goingUp = this.speedY === -10;
				const goingDown = this.speedY === 10;
				const goingRight = this.speedX === 10;
				const goingLeft = this.speedX === -10;
				if (direction === LEFT_STRING && !goingRight) {
					this.speedX = -10;
					this.speedY = 0;
				}
				if (direction === UP_STRING && !goingDown) {
					this.speedX = 0;
					this.speedY = -10;
				}
				if (direction === RIGHT_STRING && !goingLeft) {
					this.speedX = 10;
					this.speedY = 0;
				}
				if (direction === DOWN_STRING && !goingUp) {
					this.speedX = 0;
					this.speedY = 10;
				}
			}
		}
	}
</script>
