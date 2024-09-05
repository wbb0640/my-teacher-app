<template>  
  <div>  
    <h1>教师查询系统</h1>  
    <input type="text" v-model="filter" placeholder="输入课程群名字进行筛选" />  
    <ul>  
      <li v-for="teacher in filteredTeachers" :key="teacher.Name">  
        <!-- 将图片包裹在 <a> 标签内，并设置 href 属性 -->  
        <a :href="teacher.Url" target="_blank">  
          <img :src="teacher.Photo" alt="个人照片" style="width: 100px; height: auto;" />  
        </a>  
        <span> {{ teacher.Name }} - {{teacher.Department}} - {{ teacher.Title }} - {{ teacher.email }}</span>   
      </li>  
    </ul>  
  </div>  
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      teachers: [],
      filter: ""
    };
  },
  computed: {
    filteredTeachers() {
      if (this.filter) {
        return this.teachers.filter(teacher => teacher.Department.includes(this.filter));
      }
      return this.teachers;
    }
  },
  mounted() {
    axios.get("http://localhost:8000/teachers")
      .then(response => {
        this.teachers = response.data.teachers;
      })
      .catch(error => {
        console.error("Error fetching teachers:", error);
      });
  }
};
</script>

<style>
/* 添加样式 */
</style>