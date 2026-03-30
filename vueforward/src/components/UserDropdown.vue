<template>
  <div class="user-profile-container" ref="dropdownRef">
    <div class="avatar-trigger" @click="toggleDropdown">
      <div class="avatar-circle">
        {{ username ? username.charAt(0).toUpperCase() : 'U' }}
      </div>
      <span class="user-name-display">{{ username }}</span>
    </div>

    <transition name="github-fade">
      <div v-if="isOpen" class="profile-dropdown">
        <div class="user-header">
          <p class="signed-in-as">当前登录邮箱</p>
          <p class="user-email">{{ email }}</p>
        </div>
        
        <div class="dropdown-divider"></div>
        
        <div class="user-status">
          <span class="status-dot" :class="role"></span>
          {{ role === 'teacher' ? '教师权限' : '学生用户' }}
        </div>

        <div class="dropdown-divider"></div>

        <div class="menu-links">
          <a @click="handleProfile">个人设置</a>
          <a @click="handleHelp">获取帮助</a>
          <div class="dropdown-divider"></div>
          <a @click="handleLogout" class="logout-btn">退出登录</a>
        </div>
      </div>
    </transition>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, computed } from 'vue';
import { useRouter } from 'vue-router';
import axios from 'axios';

const router = useRouter();
const isOpen = ref(false);
const dropdownRef = ref(null);

const userData = ref(JSON.parse(localStorage.getItem('user') || '{}'));

const username = computed(() => userData.value.username || '未登录');
const email = computed(() => userData.value.email || '未绑定邮箱');
const role = computed(() => userData.value.role || 'student');

const updateLocalInfo = () => {
  userData.value = JSON.parse(localStorage.getItem('user') || '{}');
};

const toggleDropdown = () => {
  updateLocalInfo();
  isOpen.value = !isOpen.value;
};

const handleLogout = async () => {
  try {
    const token = localStorage.getItem('token');
    if (token) {
      await axios.post('http://127.0.0.1:33001/api/auth/logout', {}, {
        headers: { Authorization: `Bearer ${token}` }
      });
    }
  } catch (error) {
    console.error('登出接口调用失败:', error);
  } finally {
    localStorage.clear();
    isOpen.value = false;
    window.location.href = '/login';
  }
};

const handleClickOutside = (event) => {
  if (dropdownRef.value && !dropdownRef.value.contains(event.target)) {
    isOpen.value = false;
  }
};

onMounted(() => {
  updateLocalInfo();
  document.addEventListener('click', handleClickOutside);
});

onUnmounted(() => {
  document.removeEventListener('click', handleClickOutside);
});

const handleProfile = () => {
  isOpen.value = false;
  router.push('/profile');
};

const handleHelp = () => {
  alert('请联系系统管理员');
  isOpen.value = false;
};
</script>

<style scoped>
.user-profile-container {
  position: relative;
  display: inline-block;
}

.avatar-trigger {
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
  padding: 4px 8px;
  border-radius: 6px;
  transition: background 0.2s;
}

.avatar-trigger:hover {
  background: rgba(255, 255, 255, 0.1);
}

.user-name-display {
  color: #c9d1d9;
  font-size: 14px;
  font-weight: 500;
}

.avatar-circle {
  width: 32px;
  height: 32px;
  background: linear-gradient(135deg, #3b82f6, #2563eb);
  color: white;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  font-size: 14px;
  border: 1px solid rgba(255,255,255,0.2);
}

.user-email {
  font-size: 14px;
  color: #c9d1d9;
  font-weight: 600;
  margin: 4px 0 0 0;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.profile-dropdown {
  position: absolute;
  top: 45px;
  right: 0;
  width: 200px;
  background: #161b22;
  border: 1px solid #30363d;
  border-radius: 6px;
  box-shadow: 0 8px 24px rgba(0,0,0,0.5);
  z-index: 1000;
  padding: 8px 0;
}

.user-header {
  padding: 8px 16px;
}

.signed-in-as {
  font-size: 12px;
  color: #8b949e;
  margin: 0;
}

.user-status {
  padding: 8px 16px;
  font-size: 13px;
  color: #c9d1d9;
  display: flex;
  align-items: center;
  gap: 8px;
}

.status-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
}

.status-dot.teacher {
  background: #8b5cf6;
  box-shadow: 0 0 8px #8b5cf6;
}

.status-dot.student {
  background: #2ea44f;
}

.dropdown-divider {
  height: 1px;
  background: #30363d;
  margin: 8px 0;
}

.menu-links a {
  display: block;
  padding: 6px 16px;
  font-size: 14px;
  color: #c9d1d9;
  cursor: pointer;
  text-decoration: none;
}

.menu-links a:hover {
  background: #1f6feb;
  color: white;
}

.logout-btn:hover {
  background: #d5514e !important;
}

.github-fade-enter-active, .github-fade-leave-active {
  transition: opacity 0.2s, transform 0.2s;
}

.github-fade-enter-from, .github-fade-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}
</style>