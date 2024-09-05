<template>
  <div id="app">
    <img src="@/assets/logo.png" alt="北京科技大学" class="logo" />
    <div class="filters">
      <select v-model="selectedCollege" @change="fetchTeachers" class="filter-select">
        <option value="">所有学院</option>
        <option v-for="college in uniqueColleges" :key="college" :value="college">{{ college }}</option>
      </select>

      <select v-model="selectedTitle" @change="fetchTeachers" class="filter-select">
        <option value="">所有职称</option>
        <option v-for="title in uniqueTitles" :key="title" :value="title">{{ title }}</option>
      </select>

      <input
        type="text"
        v-model="nameFilter"
        placeholder="按姓名首字母检索"
        @input="fetchTeachers"
        class="filter-input"
      />
    </div>

    <div class="teacher-list">
      <div
        v-for="teacher in paginatedTeachers"
        :key="teacher.Name"
        class="teacher-card"
        @click="goToTeacherPage(teacher.Url)"
      >
        <div class="teacher-photo-container">
          <img :src="teacher.Photo" alt="教师照片" class="teacher-photo" />
          <!-- 显示飘过的评价 -->
          <transition-group name="comment-float" tag="div" class="comment-container">
            <span v-for="(comment, index) in teacher.comments" :key="index" class="comment-float">{{ comment }}</span>
          </transition-group>
        </div>
        <div class="teacher-info">
          <h2>
            {{ teacher.Name }}
            <span @click.stop="likeTeacher(teacher)" class="like-icon" :class="{ liked: teacher.liked }">♥</span>
            <span class="like-count">{{ teacher.likes }}</span>
            <span v-if="teacher.showHeart" class="heart-float" :style="{ left: `${teacher.heartX}px` }">❤️</span>
          </h2>
          <p>{{ teacher.Title }} - {{ teacher.College }}</p>
          <input
            type="text"
            v-model="teacher.comment"
            placeholder="发送评价"
            @keyup.enter="sendComment(teacher)"
            class="comment-input"
            @click.stop
          />
        </div>
      </div>
    </div>

    <div class="pagination">
      <button @click="previousPage" :disabled="!canGoBack" class="pagination-button">上一页</button>
      <span>当前页: {{ currentPage + 1 }} / {{ totalPages }}</span>
      <button @click="nextPage" :disabled="currentPage + 1 >= totalPages" class="pagination-button">下一页</button>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      teachers: [],
      selectedTitle: '',
      selectedCollege: '',
      nameFilter: '',
      currentPage: 0,
      itemsPerPage: 12,
      totalItems: 0,
      uniqueTitles: [],
      uniqueColleges: []
    };
  },
  computed: {
    totalPages() {
      return Math.ceil(this.totalItems / this.itemsPerPage);
    },
    canGoBack() {
      return this.currentPage > 0;
    },
    paginatedTeachers() {
      const start = this.currentPage * this.itemsPerPage;
      return this.teachers.slice(start, start + this.itemsPerPage);
    }
  },
  methods: {
    async fetchTeachers() {
      try {
        const response = await axios.get('http://127.0.0.1:8000/teachers/', {
          params: {
            skip: 0,
            limit: 2500,
            title: this.selectedTitle,
            college: this.selectedCollege,
            name: this.nameFilter
          }
        });
        this.teachers = response.data.teachers.map(teacher => ({
          ...teacher,
          likes: 0,
          liked: false,
          showHeart: false,
          heartX: 0,
          comment: '',
          comments: []
        }));

        this.extractUniqueTitlesAndColleges(this.teachers);
        this.totalItems = this.teachers.length;
      } catch (error) {
        console.error('Error fetching teachers:', error);
        alert('无法获取教师信息，请检查后端服务。');
      }
    },
    extractUniqueTitlesAndColleges(teachers) {
      const titlesSet = new Set();
      const collegesSet = new Set();

      teachers.forEach(teacher => {
        titlesSet.add(teacher.Title);
        collegesSet.add(teacher.College);
      });

      this.uniqueTitles = Array.from(titlesSet);
      this.uniqueColleges = Array.from(collegesSet);
    },
    likeTeacher(teacher) {
      if (!teacher.liked) {
        teacher.likes++;
        teacher.liked = true;
        this.animateHeart(teacher);
      } else {
        teacher.likes--;
        teacher.liked = false;
      }
    },
    animateHeart(teacher) {
      teacher.showHeart = true;
      teacher.heartX = Math.random() * 40 - 20;

      setTimeout(() => {
        teacher.showHeart = false;
      }, 1000);
    },
    sendComment(teacher) {
      if (teacher.comment.trim() !== '') {
        teacher.comments.push(teacher.comment);
        teacher.comment = '';

        // 设置2秒后移除评价
        setTimeout(() => {
          teacher.comments.shift();
        }, 2000);
      }
    },
    nextPage() {
      if ((this.currentPage + 1) * this.itemsPerPage < this.totalItems) {
        this.currentPage++;
        this.fetchTeachers();
      }
    },
    previousPage() {
      if (this.currentPage > 0) {
        this.currentPage--;
        this.fetchTeachers();
      }
    },
    goToTeacherPage(url) {
      window.open(url, '_blank');
    }
  },
  mounted() {
    this.fetchTeachers();
  }
};
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap');

#app {
  font-family: 'Roboto', sans-serif;
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  background-image: url('@/assets/background.jpg');
  background-size: cover;
  background-position: center;
  border-radius: 8px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
}

.logo {
  display: block;
  margin: 0 auto 20px;
  max-width: 100%;
  height: auto;
}

.filters {
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
}

.filter-select,
.filter-input {
  margin: 0 10px;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 16px;
  transition: border-color 0.3s;
}

.filter-select:focus,
.filter-input:focus {
  border-color: #007BFF;
  outline: none;
}

.teacher-list {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
}

.teacher-card {
  border: 1px solid #ccc;
  border-radius: 8px;
  padding: 10px;
  margin: 10px;
  width: calc(25% - 20px);
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  background-color: rgba(255, 255, 255, 0.8);
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s, box-shadow 0.2s;
  position: relative;
}

.teacher-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.2);
}

.teacher-photo-container {
  position: relative; /* 使评价能绝对定位于此 */
  overflow: hidden; /* 可选：确保评价不超出图片边界 */
}

.teacher-photo {
  width: 100%;
  height: auto;
  border-radius: 4px;
}

.teacher-info {
  flex-grow: 1;
}

.like-icon {
  cursor: pointer;
  margin-left: 10px;
  color: #ff0000;
  font-size: 1.2em;
}

.liked {
  color: #ff4081;
}

.like-count {
  margin-left: 5px;
}

.comment-input {
  margin-top: 10px;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
  width: calc(100% - 16px);
  font-size: 14px;
}

.pagination {
  margin-top: 20px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.pagination-button {
  padding: 10px 15px;
  margin: 0 10px;
  border: none;
  border-radius: 4px;
  background-color: #007BFF;
  color: white;
  cursor: pointer;
  transition: background-color 0.3s, transform 0.2s;
}

.pagination-button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

.pagination-button:not(:disabled):hover {
  background-color: #0056b3;
  transform: scale(1.05);
}

.heart-float {
  position: absolute;
  font-size: 2em; /* 心形大小 */
  color: red; /* 心形颜色 */
  animation: float 1s forwards; /* 添加浮动动画 */
}

@keyframes float {
  0% {
    transform: translateY(0); /* 起始位置 */
    opacity: 1; /* 完全不透明 */
  }
  100% {
    transform: translateY(-50px); /* 上升50px */
    opacity: 0; /* 渐渐透明 */
  }
}

.comment-container {
  position: absolute; /* 相对于图片定位 */
  bottom: 10px; /* 使其在图片内部 */
  left: 0; /* 左边距 */
  width: 100%; /* 宽度继承 */
  pointer-events: none; /* 确保不阻止其他事件 */
  overflow: hidden; /* 确保内容不会超出图片范围 */
}

.comment-float {
  display: inline-block; /* 让其成为行内块级元素 */
  animation: comment-float 2s forwards; /* 动画 */
}

@keyframes comment-float {
  0% {
    transform: translateX(-100%); /* 从左边位置开始 */
    opacity: 1; /* 显示 */
  }
  100% {
    transform: translateX(100%); /* 向右移动 */
    opacity: 0; /* 消失 */
  }
}
</style>