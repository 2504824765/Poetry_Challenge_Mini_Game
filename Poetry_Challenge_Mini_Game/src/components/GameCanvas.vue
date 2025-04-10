<template>
    <div class="game-container">
      <canvas ref="gameCanvas" :width="canvasWidth" :height="canvasHeight"></canvas>
      
      <div v-if="gameOver" class="game-over">
        <h2>游戏结束</h2>
        <p>最终得分: {{ score }}</p>
        <button @click="restartGame">重新开始</button>
      </div>
      
      <div v-if="showQuestion" class="question-modal">
        <h3>{{ currentQuestion.question }}</h3>
        <div class="options">
          <button 
            v-for="(option, index) in currentQuestion.options" 
            :key="index"
            @click="checkAnswer(option.answer)"
            :disabled="answerSubmitted"
          >
            {{ option.text }}
          </button>
        </div>
        <div v-if="answerSubmitted" class="feedback" :class="answerCorrect ? 'correct' : 'incorrect'">
          {{ answerFeedback }}
        </div>
      </div>
      
      <div class="score-display">得分: {{ score }}</div>
      <div class="level-display">关卡: {{ currentLevel + 1 }}</div>
    </div>
  </template>
  
  <script>
  export default {
    name: 'GameCanvas',
    data() {
      return {
        canvas: null,
        ctx: null,
        canvasWidth: 1200,  // 加宽画布
        canvasHeight: 500,
        player: {
          x: 50,
          y: 400,
          width: 30,
          height: 50,
          velocityY: 0,
          jumping: false,
          speed: 5,
          color: '#3498db'
        },
        // 多关卡设计
        levels: [
          // 第一关
          {
            obstacles: [
              { x: 300, y: 450, width: 200, height: 50, color: '#e74c3c' },
              { x: 600, y: 350, width: 75, height: 150, color: '#e74c3c' }
            ],
            rewards: [
              { x: 400, y: 380, width: 20, height: 20, color: '#2ecc71', collected: false },
              { x: 700, y: 300, width: 20, height: 20, color: '#2ecc71', collected: false },
              { x: 900, y: 400, width: 20, height: 20, color: '#2ecc71', collected: false }
            ],
            endPoint: { x: 1100, y: 400, width: 50, height: 50, color: '#f39c12' }
          },
          // 第二关 (更难)
          {
            obstacles: [
              { x: 250, y: 450, width: 150, height: 50, color: '#e74c3c' },
              { x: 450, y: 350, width: 75, height: 110, color: '#e74c3c' },
              { x: 650, y: 400, width: 80, height: 100, color: '#e74c3c' },
              { x: 800, y: 300, width: 50, height: 100, color: '#e74c3c' }
            ],
            rewards: [
              { x: 350, y: 380, width: 20, height: 20, color: '#2ecc71', collected: false },
              { x: 550, y: 300, width: 20, height: 20, color: '#2ecc71', collected: false },
              { x: 750, y: 350, width: 20, height: 20, color: '#2ecc71', collected: false },
              { x: 950, y: 250, width: 20, height: 20, color: '#2ecc71', collected: false }
            ],
            endPoint: { x: 1100, y: 400, width: 50, height: 50, color: '#f39c12' }
          },
          // 第三关 (最难)
          {
            obstacles: [
              { x: 200, y: 450, width: 100, height: 50, color: '#e74c3c' },
              { x: 350, y: 400, width: 70, height: 100, color: '#e74c3c' },
              { x: 500, y: 350, width: 60, height: 110, color: '#e74c3c' },
              { x: 650, y: 300, width: 60, height: 100, color: '#e74c3c' },
              { x: 800, y: 400, width: 70, height: 80, color: '#e74c3c' },
              { x: 950, y: 350, width: 80, height: 110, color: '#e74c3c' }
            ],
            rewards: [
              { x: 300, y: 380, width: 20, height: 20, color: '#2ecc71', collected: false },
              { x: 450, y: 330, width: 20, height: 20, color: '#2ecc71', collected: false },
              { x: 600, y: 280, width: 20, height: 20, color: '#2ecc71', collected: false },
              { x: 750, y: 350, width: 20, height: 20, color: '#2ecc71', collected: false },
              { x: 900, y: 320, width: 20, height: 20, color: '#2ecc71', collected: false }
            ],
            endPoint: { x: 1100, y: 400, width: 50, height: 50, color: '#f39c12' }
          }
        ],
        currentLevel: 0,
        keys: {
          left: false,
          right: false
        },
        score: 0,
        gameOver: false,
        showQuestion: false,
        currentQuestion: null,
        answerSubmitted: false,
        answerCorrect: false,
        answerFeedback: '',
        questions: [
          {
            question: "\"床前明月光\"的下一句是什么？",
            options: [
              { text: "A. 疑是地上霜", answer: "A" },
              { text: "B. 举头望明月", answer: "B" },
              { text: "C. 低头思故乡", answer: "C" },
              { text: "D. 明月几时有", answer: "D" }
            ],
            correctAnswer: "A",
            explanation: "正确答案是A，出自李白《静夜思》"
          },
          {
            question: "\"春眠不觉晓\"的下一句是什么？",
            options: [
              { text: "A. 夜来风雨声", answer: "A" },
              { text: "B. 花落知多少", answer: "B" },
              { text: "C. 处处闻啼鸟", answer: "C" },
              { text: "D. 白日依山尽", answer: "D" }
            ],
            correctAnswer: "C",
            explanation: "正确答案是C，出自孟浩然《春晓》"
          },
          {
            question: "李白《将进酒》中\"天生我材必有用\"的下一句是什么？",
            options: [
              { text: "A. 千金散尽还复来", answer: "A" },
              { text: "B. 会须一饮三百杯", answer: "B" },
              { text: "C. 与尔同销万古愁", answer: "C" },
              { text: "D. 但愿长醉不复醒", answer: "D" }
            ],
            correctAnswer: "A",
            explanation: "正确答案是A，出自李白《将进酒》"
          },
          {
            question: "杜甫《春望》中\"国破山河在\"的下一句是什么？",
            options: [
              { text: "A. 城春草木深", answer: "A" },
              { text: "B. 感时花溅泪", answer: "B" },
              { text: "C. 烽火连三月", answer: "C" },
              { text: "D. 家书抵万金", answer: "D" }
            ],
            correctAnswer: "A",
            explanation: "正确答案是A，出自杜甫《春望》"
          },
          {
            question: "苏轼《水调歌头》中\"明月几时有\"的下一句是什么？",
            options: [
              { text: "A. 把酒问青天", answer: "A" },
              { text: "B. 不知天上宫阙", answer: "B" },
              { text: "C. 今夕是何年", answer: "C" },
              { text: "D. 起舞弄清影", answer: "D" }
            ],
            correctAnswer: "A",
            explanation: "正确答案是A，出自苏轼《水调歌头》"
          }
        ],
        animationFrameId: null,
        gravity: 0.5,
        jumpForce: -12,
        cameraOffset: 0
      }
    },
    computed: {
      currentObstacles() {
        return this.levels[this.currentLevel].obstacles
      },
      currentRewards() {
        return this.levels[this.currentLevel].rewards
      },
      currentEndPoint() {
        return this.levels[this.currentLevel].endPoint
      }
    },
    mounted() {
      this.canvas = this.$refs.gameCanvas
      this.ctx = this.canvas.getContext('2d')
      
      // 设置事件监听
      window.addEventListener('keydown', this.handleKeyDown)
      window.addEventListener('keyup', this.handleKeyUp)
      
      // 开始游戏循环
      this.gameLoop()
    },
    beforeUnmount() {
      // 清除事件监听和动画帧
      window.removeEventListener('keydown', this.handleKeyDown)
      window.removeEventListener('keyup', this.handleKeyUp)
      cancelAnimationFrame(this.animationFrameId)
    },
    methods: {
      handleKeyDown(e) {
        if (this.gameOver || this.showQuestion) return
        
        switch(e.key) {
          case 'ArrowLeft':
            this.keys.left = true
            break
          case 'ArrowRight':
            this.keys.right = true
            break
          case ' ':
            if (!this.player.jumping) {
              this.player.velocityY = this.jumpForce
              this.player.jumping = true
            }
            break
        }
      },
      handleKeyUp(e) {
        switch(e.key) {
          case 'ArrowLeft':
            this.keys.left = false
            break
          case 'ArrowRight':
            this.keys.right = false
            break
        }
      },
      gameLoop() {
        this.update()
        this.render()
        this.animationFrameId = requestAnimationFrame(this.gameLoop)
      },
      update() {
        if (this.gameOver || this.showQuestion) return
        
        // 玩家移动
        if (this.keys.left) {
          this.player.x -= this.player.speed
          if (this.player.x < 0) this.player.x = 0
        }
        if (this.keys.right) {
          this.player.x += this.player.speed
          if (this.player.x > this.canvasWidth - this.player.width) {
            this.player.x = this.canvasWidth - this.player.width
          }
        }
        
        // 重力
        this.player.velocityY += this.gravity
        this.player.y += this.player.velocityY
        
        // 地面检测
        if (this.player.y >= 400) {
          this.player.y = 400
          this.player.velocityY = 0
          this.player.jumping = false
        }
        
        // 障碍物碰撞检测
        for (const obstacle of this.currentObstacles) {
          if (this.checkCollision(this.player, obstacle)) {
            this.gameOver = true
            break
          }
        }
        
        // 奖励物品碰撞检测
        for (const reward of this.currentRewards) {
          if (!reward.collected && this.checkCollision(this.player, reward)) {
            reward.collected = true
            this.showRandomQuestion()
            break
          }
        }
        
        // 终点检测
        if (this.checkCollision(this.player, this.currentEndPoint)) {
          if (this.currentLevel < this.levels.length - 1) {
            this.nextLevel()
          } else {
            // 通关
            this.gameOver = true
          }
        }
        
        // 简单的相机跟随
        if (this.player.x > this.canvasWidth / 2) {
          this.cameraOffset = Math.min(
            this.player.x - this.canvasWidth / 2,
            this.canvasWidth * 2 - this.canvasWidth
          )
        } else {
          this.cameraOffset = 0
        }
      },
      render() {
        // 清空画布
        this.ctx.clearRect(0, 0, this.canvasWidth, this.canvasHeight)
        
        // 绘制地面
        this.ctx.fillStyle = '#34495e'
        this.ctx.fillRect(0 - this.cameraOffset, 450, this.canvasWidth * 2, this.canvasHeight - 450)
        
        // 绘制玩家
        this.ctx.fillStyle = this.player.color
        this.ctx.fillRect(
          this.player.x - this.cameraOffset, 
          this.player.y, 
          this.player.width, 
          this.player.height
        )
        
        // 绘制障碍物
        this.ctx.fillStyle = '#e74c3c'
        for (const obstacle of this.currentObstacles) {
          this.ctx.fillRect(
            obstacle.x - this.cameraOffset, 
            obstacle.y, 
            obstacle.width, 
            obstacle.height
          )
        }
        
        // 绘制奖励物品
        this.ctx.fillStyle = '#2ecc71'
        for (const reward of this.currentRewards) {
          if (!reward.collected) {
            this.ctx.fillRect(
              reward.x - this.cameraOffset, 
              reward.y, 
              reward.width, 
              reward.height
            )
          }
        }
        
        // 绘制终点
        this.ctx.fillStyle = this.currentEndPoint.color
        this.ctx.fillRect(
          this.currentEndPoint.x - this.cameraOffset,
          this.currentEndPoint.y,
          this.currentEndPoint.width,
          this.currentEndPoint.height
        )
        
        // 绘制终点旗帜
        this.ctx.fillStyle = '#f1c40f'
        this.ctx.beginPath()
        this.ctx.moveTo(
          this.currentEndPoint.x - this.cameraOffset + this.currentEndPoint.width,
          this.currentEndPoint.y
        )
        this.ctx.lineTo(
          this.currentEndPoint.x - this.cameraOffset + this.currentEndPoint.width + 20,
          this.currentEndPoint.y + this.currentEndPoint.height / 2
        )
        this.ctx.lineTo(
          this.currentEndPoint.x - this.cameraOffset + this.currentEndPoint.width,
          this.currentEndPoint.y + this.currentEndPoint.height
        )
        this.ctx.closePath()
        this.ctx.fill()
      },
      checkCollision(obj1, obj2) {
        return obj1.x < obj2.x + obj2.width &&
               obj1.x + obj1.width > obj2.x &&
               obj1.y < obj2.y + obj2.height &&
               obj1.y + obj1.height > obj2.y
      },
      showRandomQuestion() {
        this.currentQuestion = this.questions[Math.floor(Math.random() * this.questions.length)]
        this.showQuestion = true
        this.answerSubmitted = false
      },
      checkAnswer(selectedAnswer) {
        this.answerSubmitted = true
        this.answerCorrect = selectedAnswer === this.currentQuestion.correctAnswer
        this.answerFeedback = this.answerCorrect 
          ? "回答正确！+" + 10 + "分\n" + this.currentQuestion.explanation
          : "回答错误！\n" + this.currentQuestion.explanation
        
        if (this.answerCorrect) {
          this.score += 10
        }
        
        // 2秒后自动关闭问题框
        setTimeout(() => {
          this.showQuestion = false
        }, 2000)
      },
      nextLevel() {
        this.currentLevel++
        this.player.x = 50
        this.player.y = 400
        this.player.velocityY = 0
        this.player.jumping = false
        this.cameraOffset = 0
        
        // 重置奖励物品
        for (const reward of this.currentRewards) {
          reward.collected = false
        }
      },
      restartGame() {
        this.currentLevel = 0
        this.player.x = 50
        this.player.y = 400
        this.player.velocityY = 0
        this.player.jumping = false
        this.cameraOffset = 0
        
        this.score = 0
        this.gameOver = false
        
        // 重置奖励物品
        for (const reward of this.currentRewards) {
          reward.collected = false
        }
      }
    }
  }
  </script>
  
  <style scoped>
  .game-container {
    position: relative;
    width: 100%;
    max-width: 1200px;
    height: 500px;
    background-color: #f0f0f0;
    border: 2px solid #333;
    overflow: hidden;
    margin: 0 auto;
  }
  
  .game-over {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    background-color: rgba(255, 255, 255, 0.9);
    padding: 20px;
    border-radius: 10px;
    text-align: center;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.3);
    z-index: 10;
    width: 80%;
    max-width: 400px;
  }
  
  .game-over h2 {
    margin-top: 0;
    color: #e74c3c;
  }
  
  .game-over button {
    padding: 10px 20px;
    background-color: #3498db;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 16px;
    margin-top: 10px;
  }
  
  .game-over button:hover {
    background-color: #2980b9;
  }
  
  .question-modal {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    background-color: rgba(255, 255, 255, 0.95);
    padding: 20px;
    border-radius: 10px;
    width: 90%;
    max-width: 500px;
    box-shadow: 0 0 15px rgba(0, 0, 0, 0.2);
    z-index: 10;
  }
  
  .question-modal h3 {
    margin-top: 0;
    color: #2c3e50;
    font-size: 18px;
  }
  
  .options {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 10px;
    margin: 20px 0;
  }
  
  .options button {
    padding: 12px;
    background-color: #3498db;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 14px;
    transition: background-color 0.2s;
  }
  
  .options button:hover:not(:disabled) {
    background-color: #2980b9;
  }
  
  .options button:disabled {
    opacity: 0.7;
    cursor: not-allowed;
  }
  
  .feedback {
    padding: 10px;
    border-radius: 4px;
    margin-top: 10px;
    font-size: 14px;
    white-space: pre-line;
  }
  
  .feedback.correct {
    background-color: rgba(46, 204, 113, 0.2);
    color: #27ae60;
    border: 1px solid #2ecc71;
  }
  
  .feedback.incorrect {
    background-color: rgba(231, 76, 60, 0.2);
    color: #c0392b;
    border: 1px solid #e74c3c;
  }
  
  .score-display {
    position: absolute;
    top: 10px;
    left: 10px;
    font-size: 20px;
    color: #2c3e50;
    background-color: rgba(255, 255, 255, 0.7);
    padding: 5px 10px;
    border-radius: 4px;
  }
  
  .level-display {
    position: absolute;
    top: 10px;
    right: 10px;
    font-size: 20px;
    color: #2c3e50;
    background-color: rgba(255, 255, 255, 0.7);
    padding: 5px 10px;
    border-radius: 4px;
  }
  </style>