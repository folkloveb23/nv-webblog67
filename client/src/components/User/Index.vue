<template>
    <div class="container">
    <h1>ผู้ใช้งาน</h1>
    <div class="header-actions">
      <button>class="btn success" v-on:click="navigateTo('/user/create')">Register</button>
    </div>
  </div>
    <hr>
    <div v-if="users.length">
      <div class="user-count"><b>จำนวนผู้ใช้งาน:</b> {{ users.length }}</div>
      <div v-for="user in users" v-bind:key="user.id" class="user-card">
        <div><b>ID:</b> {{ user.id }}</div>
        <div><b>ชื่อผู้ใช้:</b> {{ user.name }} {{ user.lastname }}</div>
        <div><b>อีเมล:</b> {{ user.email }}</div>
        <div><b>สถานะ:</b> {{ user.status }}</div>
        <div><b>ประเภท:</b> {{ user.type }}</div>
        <div class="actions">
          <button class="btn info" v-on:click="navigateTo('/user/'+user.id)">ดูข้อมูล</button>
          <button class="btn warning" v-on:click="navigateTo('/user/edit/'+user.id)">แก้ไขข้อมูล</button>
          <button class="btn danger" v-on:click="deleteUser(user)">ลบข้อมูล</button>
        <hr>
      </div>
    </div>
    <div class="footer-actions">
      <button class="btn logout" v-on:click="logout">Logout</button>
    </div>
  </div>
</template>

<script>
import UsersService from "@/services/UsersService";
export default {
  data(){
    return {
      users: []
    }
  },
  async created() {
    try{
      this.users = (await UsersService.index()).data;
    }catch(err){
      console.log(err);
    }
  },
  methods:{
    logout(){
      this.$store.dispatch('setToken',null)
      this.$store.dispatch('setUser',null)
      this.$router.push({
        name: 'login'
      })
    },
    navigateTo(route){
      this.$router.push(route);
    },
    async deleteUser(user){
      let result = confirm("คุณต้องการลบข้อมูลใช่หรือไม่?");
      if(result){
        try{
          await UsersService.delete(user);
          this.refreshData();

        }catch(err){
          console.log(err);
        }
      }
      methods: {
    logout(); {
      this.$store.dispatch('setToken', null)
      this.$store.dispatch('setUser', null)
      this.$router.push({ name: 'login' })}
      }
    },
    async refreshData(){
      try{
        this.users = (await UsersService.index()).data;
      }catch(err){
        console.log(err);
      }
    }
  }
};
</script>

<style scoped>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  background-color: #fff5f5;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}
h1 {
  text-align: center;
  color: #b71c1c;
  margin-bottom: 20px;
}
.header-actions, .footer-actions {
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
}
.user-count {
  font-size: 18px;
  color: #333;
  margin-bottom: 10px;
}
.user-card {
  background-color: #ffffff;
  padding: 15px;
  border-radius: 8px;
  margin-bottom: 20px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}
.actions {
  display: flex;
  gap: 10px;
  margin-top: 10px;
}
hr {
  border: none;
  border-top: 1px solid #ffcdd2;
  margin: 15px 0;
}
.btn {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  color: #fff;
  cursor: pointer;
  transition: background-color 0.3s ease;
}
.btn.success {
  background-color: #d32f2f;
}
.btn.success:hover {
  background-color: #b71c1c;
}
.btn.info {
  background-color: #64b5f6;
}
.btn.info:hover {
  background-color: #42a5f5;
}
.btn.warning {
  background-color: #ffb74d;
}
.btn.warning:hover {
  background-color: #ffa726;
}
.btn.danger {
  background-color: #e57373;
}
.btn.danger:hover {
  background-color: #ef5350;
}
.btn.logout {
  background-color: #d32f2f;
}
.btn.logout:hover {
  background-color: #b71c1c;
}
</style>
