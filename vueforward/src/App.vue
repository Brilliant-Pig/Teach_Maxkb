<template>
  <div id="app">
    <div class="particles-bg"></div>

    <nav v-if="showNav" class="main-nav animate__animated animate__fadeInDown">
      <div class="nav-logo">OS-AI 智学系统</div>
      
      <div class="nav-links">
        <router-link to="/ImmersiveLab" class="nav-link">AI实验室</router-link>
        <router-link to="/HomeworkAssistant" class="nav-link">作业助手</router-link>
        <router-link to="/LearningDashboard" class="nav-link">学业评价</router-link>
        
        <router-link 
          v-if="isLoggedIn && userRole === 'teacher'" 
          to="/TeacherConsole" 
          class="nav-link"
        >教师后台</router-link>

        <UserAvatar v-if="isLoggedIn" />
      </div>
    </nav>
    
    <div class="view-container">
      <router-view v-slot="{ Component }">
        <transition name="fade-transform" mode="out-in">
          <component :is="Component" />
        </transition>
      </router-view>
    </div>

    <footer v-if="showNav" class="app-footer">
      <div class="footer-content">
        <span>当前环境: 实验室私有云 (RTX 4060)</span>
        <span class="divider">|</span>
        <span>推理引擎: Ollama / Qwen2</span>
        <span class="divider">|</span>
        <span class="security-tag">🛡️ 数据不出校 · 安全加密中</span>
      </div>
    </footer>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue'; 
import { useRoute } from 'vue-router';
import UserAvatar from './components/UserDropdown.vue';

const route = useRoute();
const showNav = ref(true);
const isLoggedIn.ref(false);
const userRole.ref('');

const updateAuthState = () => {
  try {
    const token = localStorage.getItem('token');
    const userStr = localStorage.getItem('user');
    
    if (token && userStr && userStr !== 'undefined') {
      const user = JSON.parse(userStr);
      isLoggedIn.value = true;
      userRole.value = user.role || '';
    } else {
      isLoggedIn.value = false;
      userRole.value = '';
    }
  } catch (e) {
    console.error("解析用户信息失败", e);
    isLoggedIn.value = false;
  }
};

onMounted(() => {
  updateAuthState();

  // 🎆 全局点击烟花特效
  document.addEventListener("click", (e) => {
    if (e.target.closest('.user-profile-container')) {
      return; 
    }

    const x = e.clientX;
    const y = e.clientY;

    const colors = [
      "#409eff", "#71b7ff", "#a29bfe", "#6c5ce7",
      "#00c48c", "#ff7675", "#fd79a8", "#ffeaa7"
    ];

    for (let i = 0; i < 24; i++) {
      createParticle(x, y, colors);
    }
  });

  function createParticle(x, y, colors) {
    const particle = document.createElement("div");
    particle.classList.add("firework");
    particle.style.left = x + "px";
    particle.style.top = y + "px";
    particle.style.background = colors[Math.floor(Math.random() * colors.length)];

    const angle = Math.random() * Math.PI * 2;
    const speed = 2 + Math.random() * 7;
    const vx = Math.cos(angle) * speed;
    const vy = Math.sin(angle) * speed;
    const drag = 0.95;

    let px = 0, py = 0;
    let cvx = vx, cvy = vy;

    const animate = () => {
      px += cvx;
      py += cvy;
      cvx *= drag;
      cvy *= drag;

      particle.style.transform = `translate(${px}px, ${py}px)`;

      if (Math.abs(cvx) < 0.1 && Math.abs(cvy) < 0.1) {
        particle.remove();
        return;
      }
      requestAnimationFrame(animate);
    };

    document.body.appendChild(particle);
    requestAnimationFrame(animate);
  }
});
});

watch(() => route.path, (newPath) => {
  showNav.value = !['/login', '/Auth'].includes(newPath);
  updateAuthState();
});
</script>

<style>
/* 基础重置 */
body, html {
  margin: 0;
  padding: 0;
  height: 100%;
  font-family: 'PingFang SC', 'Microsoft YaHei', sans-serif;
  overflow: hidden;
  background: #0a0e14;
}

#app {
  height: 100vh;
  display: flex;
  flex-direction: column;
  position: relative;
  color: #e0e0e0;
}


.firework {
  position: fixed;
  width: 5px;
  height: 5px;
  border-radius: 50%;
  pointer-events: none;
  z-index: 999999;
  box-shadow: 0 0 6px currentColor;
  animation: firework-fade 1.1s ease-out forwards;
}

@keyframes firework-fade {
  0% { opacity: 1; transform: scale(1); }
  100% { opacity: 0; transform: scale(2.8); }
}

/* 粒子背景动画 */
.particles-bg {
  position: absolute;
  inset: 0;
  z-index: -1;
  background: radial-gradient(circle at center, #1b2735 0%, #090a0f 100%);
  overflow: hidden;
}

.particles-bg::after {
  content: "";
  position: absolute;
  width: 2px;
  height: 2px;
  background: white;
  box-shadow: 
    10vw 20vh 1px #fff, 30vw 50vh 1px #fff, 70vw 10vh 2px #fff,
    90vw 80vh 1px #fff, 40vw 90vh 2px #fff, 15vw 40vh 1px #fff;
  border-radius: 50%;
  opacity: 0.3;
  animation: bg-move 100s linear infinite;
}

@keyframes bg-move {
  from { transform: translateY(0); }
  to { transform: translateY(-1000px); }
}

/* 导航样式 */
.main-nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 40px;
  height: 64px;
  background: rgba(255, 255, 255, 0.05);
  backdrop-filter: blur(10px);
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
  z-index: 100;
}

.nav-logo {
  font-size: 1.4rem;
  font-weight: bold;
  background: linear-gradient(120deg, #409eff, #71b7ff);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.nav-links {
  display: flex;
  gap: 30px;
  align-items: center;
}

.nav-link {
  text-decoration: none;
  color: #909399;
  font-size: 1rem;
  transition: all 0.3s;
}

.nav-link:hover, .router-link-active {
  color: #409eff;
  text-shadow: 0 0 10px rgba(64, 158, 255, 0.5);
}

/* 内容容器 */
.view-container {
  flex: 1;
  overflow: hidden;
  position: relative;
}

/* 页面切换动画 */
.fade-transform-enter-active,
.fade-transform-leave-active {
  transition: all 0.4s ease;
}

.fade-transform-enter-from {
  opacity: 0;
  transform: translateX(-30px);
}

.fade-transform-leave-to {
  opacity: 0;
  transform: translateX(30px);
}

/* 页脚 */
.app-footer {
  height: 32px;
  background: rgba(0, 0, 0, 0.3);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 12px;
  color: #606266;
  border-top: 1px solid rgba(255, 255, 255, 0.05);
}

.divider {
  margin: 0 15px;
  opacity: 0.3;
}

.security-tag {
  color: #67c23a;
}
</style>