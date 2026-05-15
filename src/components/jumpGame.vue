<template>
  <div class="game-wrapper">
    <!-- Score Display -->
    <div class="score-board">Score: {{ score }}</div>

    <!-- Game Container -->
    <div class="game-container">
      <!-- Player Element -->
      <div 
        class="player" 
        :class="{ 'jump-animation': isJumping }"
        ref="playerRef"
      ></div>

      <!-- Obstacle Element -->
      <div 
        class="obstacle" 
        :style="{ left: obstacleX + 'px' }"
        ref="obstacleRef"
      ></div>

      <!-- Game Over Overlay -->
      <div v-if="gameOver" class="game-over-overlay">
        <h2>Game Over</h2>
        <button @click="resetGame">Restart Game</button>
      </div>
    </div>
    <p class="controls-hint">Press <strong>Spacebar</strong> to Jump & Start Music</p>
  </div>
</template>

<script>
export default {
  name: "JumpGame",
  data() {
    return {
      score: 0,
      gameOver: false,
      isJumping: false,
      obstacleX: 600,
      speed: 6,
      animationFrameId: null,
      bgMusic: null,
      isMusicPlaying: false,
      
      // VUE CLI / WEBPACK FIX: Use require() with the @ alias to resolve paths accurately
      jumpSoundSrc: require('@/assets/sound/Jump.mp3'),
      pointSoundSrc: require('@/assets/sound/Point.mp3'),
      bgMusicSrc: require('@/assets/sound/background.mp3')
    };
  },
  mounted() {
    window.addEventListener("keydown", this.handleKeyDown);
    this.startGame();
  },
  unmounted() {
    window.removeEventListener("keydown", this.handleKeyDown);
    cancelAnimationFrame(this.animationFrameId);
    this.stopMusic();
  },
  methods: {
    startGame() {
      this.gameLoop();
    },
    handleKeyDown(event) {
      if (event.code === "Space") {
        event.preventDefault(); 
        
        if (this.gameOver) return;

        // Start background music loop on the first user interaction
        if (!this.isMusicPlaying) {
          this.playMusic();
        }
        
        if (!this.isJumping) {
          this.jump();
        }
      }
    },
    jump() {
      this.isJumping = true;
      this.playSound('jump');

      setTimeout(() => {
        this.isJumping = false;
      }, 600);
    },
    gameLoop() {
      if (this.gameOver) return;
      this.obstacleX -= this.speed;

      if (this.obstacleX < -20) {
        this.obstacleX = 600;
        this.incrementScore();
      }

      this.checkCollision();

      if (!this.gameOver) {
        this.animationFrameId = requestAnimationFrame(this.gameLoop);
      }
    },
    checkCollision() {
      const player = this.$refs.playerRef;
      const obstacle = this.$refs.obstacleRef;
      if (!player || !obstacle) return;

      const playerRect = player.getBoundingClientRect();
      const obstacleRect = obstacle.getBoundingClientRect();

      const hasHorizontalOverlap = obstacleRect.left < playerRect.right && obstacleRect.right > playerRect.left;
      const hasVerticalOverlap = obstacleRect.top < playerRect.bottom && obstacleRect.bottom > playerRect.top;

      if (hasHorizontalOverlap && hasVerticalOverlap) {
        this.gameOver = true;
        cancelAnimationFrame(this.animationFrameId);
        this.stopMusic(); 
      }
    },
    incrementScore() {
      this.score += 1;
      this.playSound('score');

      if (this.score % 4 === 0) {
        this.speed += 1;
      }
    },
    playSound(soundName) {
      let targetSrc = "";

      if (soundName === 'jump') {
        targetSrc = this.jumpSoundSrc;
      } else if (soundName === 'score') {
        targetSrc = this.pointSoundSrc;
      }

      if (targetSrc) {
        const audio = new Audio(targetSrc);
        audio.volume = 0.4;
        audio.play().catch(err => console.log("SFX playback blocked:", err));
      }
    },
    playMusic() {
      this.bgMusic = new Audio(this.bgMusicSrc);
      this.bgMusic.loop = true;
      this.bgMusic.volume = 1;
      this.bgMusic.play()
        .then(() => {
          this.isMusicPlaying = true;
        })
        .catch(err => console.log("BGM playback blocked:", err));
    },
    stopMusic() {
      if (this.bgMusic) {
        this.bgMusic.pause();
        this.bgMusic.currentTime = 0;
        this.isMusicPlaying = false;
      }
    },
    resetGame() {
      this.score = 0;
      this.gameOver = false;
      this.isJumping = false;
      this.obstacleX = 600;
      this.speed = 6;
      this.playMusic(); 
      this.startGame();
    }
  }
};
</script>

<style scoped>
.game-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  font-family: Arial, sans-serif;
  margin-top: 20px;
}

.score-board {
  font-size: 24px;
  font-weight: bold;
  margin-bottom: 10px;
}

.game-container {
  position: relative;
  width: 600px;
  height: 200px;
  background-color: #f0f0f0;
  border: 2px solid #333;
  overflow: hidden;
  border-radius: 4px;
}

.player {
  position: absolute;
  bottom: 0;
  left: 50px;
  width: 40px;
  height: 40px;
  background-color: #3498db;
  border-radius: 4px;
}

.jump-animation {
  animation: jumpArc 0.6s cubic-bezier(0.25, 0.46, 0.45, 0.94) forwards;
}

@keyframes jumpArc {
  0% { bottom: 0; }
  50% { bottom: 110px; }
  100% { bottom: 0; }
}

.obstacle {
  position: absolute;
  bottom: 0;
  width: 20px;
  height: 40px;
  background-color: #e74c3c;
  border-radius: 2px;
}

.game-over-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.8);
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  color: white;
  z-index: 10;
}

.game-over-overlay h2 {
  margin: 0 0 15px 0;
  font-size: 24px;
  color: #e74c3c;
}

.game-over-overlay button {
  padding: 10px 20px;
  font-size: 16px;
  cursor: pointer;
  background-color: #2ecc71;
  border: none;
  color: white;
  border-radius: 4px;
  font-weight: bold;
}

.game-over-overlay button:hover {
  background-color: #27ae60;
}

.controls-hint {
  margin-top: 10px;
  color: #666;
}
</style>
