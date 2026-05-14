<template>
  <div class="game-wrapper">
    <!-- Score Display -->
    <div class="score-board">Score: {{ score }}</div>

    <!-- Game Container -->
    <div class="game-container">
      <!-- Player Element: Class 'jump-animation' is applied ONLY during a jump -->
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
    <p class="controls-hint">Press <strong>Spacebar</strong> once to Jump</p>
  </div>
</template>

<script>
export default {
  name: "JumpGame",
  data() {
    return {
      score: 0,
      gameOver: false,
      isJumping: false,   // Tracks if jump is in progress
      obstacleX: 600,     // Starting position at right edge
      speed: 6,           // Constant horizontal movement speed
      animationFrameId: null
    };
  },
  mounted() {
    // Listen for keyboard inputs globally
    window.addEventListener("keydown", this.handleKeyDown);
    // Start the game loop immediately
    this.startGame();
  },
  unmounted() {
    // Clean up to prevent background leaks
    window.removeEventListener("keydown", this.handleKeyDown);
    cancelAnimationFrame(this.animationFrameId);
  },
  methods: {
    startGame() {
      this.gameLoop();
    },
    handleKeyDown(event) {
      if (event.code === "Space") {
        event.preventDefault(); // Stop page from scrolling down
        
        // CRITICAL FIX: If already jumping or game is over, ignore press entirely
        if (this.isJumping || this.gameOver) return;
        
        this.jump();
      }
    },
    jump() {
      this.isJumping = true;

      // The jump animation lasts exactly 600ms (defined in CSS below)
      // Once 600ms passes, the character lands, allowing another jump
      setTimeout(() => {
        this.isJumping = false;
      }, 600);
    },
    gameLoop() {
      if (this.gameOver) return;

      // Move obstacle left by speed value
      this.obstacleX -= this.speed;

      // Reset obstacle when it leaves the left side of the box (-20px width)
      if (this.obstacleX < -20) {
        this.obstacleX = 600;
        this.incrementScore();
      }

      // Check collision coordinates on every single frame
      this.checkCollision();

      if (!this.gameOver) {
        this.animationFrameId = requestAnimationFrame(this.gameLoop);
      }
    },
    checkCollision() {
      const player = this.$refs.playerRef;
      const obstacle = this.$refs.obstacleRef;

      if (!player || !obstacle) return;

      // Get precise, real-time bounding boxes directly from the screen pixels
      const playerRect = player.getBoundingClientRect();
      const obstacleRect = obstacle.getBoundingClientRect();

      // Math formulas to detect any physical pixel boundary intersections
      const hasHorizontalOverlap = obstacleRect.left < playerRect.right && obstacleRect.right > playerRect.left;
      const hasVerticalOverlap = obstacleRect.top < playerRect.bottom && obstacleRect.bottom > playerRect.top;

      if (hasHorizontalOverlap && hasVerticalOverlap) {
        this.gameOver = true;
        cancelAnimationFrame(this.animationFrameId);
      }
    },
    incrementScore() {
      this.score += 1;
      // Increase difficulty slightly every 4 points
      if (this.score % 4 === 0) {
        this.speed += 1;
      }
    },
    resetGame() {
      this.score = 0;
      this.gameOver = false;
      this.isJumping = false;
      this.obstacleX = 600;
      this.speed = 6;
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

/* This smooth animation forces the player to come down automatically */
.jump-animation {
  animation: jumpArc 0.6s cubic-bezier(0.25, 0.46, 0.45, 0.94) forwards;
}

@keyframes jumpArc {
  0% {
    bottom: 0;
  }
  50% {
    bottom: 110px; /* Peak height of jump */
  }
  100% {
    bottom: 0; /* Forced landing */
  }
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
}

.game-over-overlay h2 {
  margin: 0 0 15px 0;
  font-size: 28px;
  color: #e74c3c;
}

.game-over-overlay button {
  padding: 8px 16px;
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
