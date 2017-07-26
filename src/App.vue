<script>
import photos from './components/photos';

export default {
  name: 'app',
  components: {
    'photos': photos
  },
  data(){
    return {
      authorized: false,
      scope: 'email, public_profile, user_photos',
      profile: {}
    }
  },
  computed: {
    profilePicture(){
      return `https://graph.facebook.com/${this.profile.id}/picture?type=large`;
    }
  },
  created(){
    let vm = this;
    window.checkLoginState = function(){
      FB.getLoginStatus(response => {
        vm.statusChangeCallback(response);
      });
    }
  },
  mounted(){
    window.fbAsyncInit = function() {
      FB.init({
        appId      : '1443327865754667',
        cookie     : true,
        xfbml      : true,
        version    : 'v2.10'
      });
      FB.AppEvents.logPageView();

      // 不 check 的話 getProfile 會 get 不到
      checkLoginState();
    };
  },
  methods: {
    getProfile(){
      FB.api('/me?fields=name,id,email', response => {
        this.$set(this, 'profile', response)
      });
    },
    statusChangeCallback(response){
      if (response.status === 'connected'){
        this.authorized = true;
        this.getProfile();
      }
      else if (response.status === 'not_authorized'){
        this.authorized = false;
      }
      else {
        this.authorized = false;
      }
    }
  },
}
</script>

<style lang="scss">
#app {
  font-size: 18px;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;

  .profile{
    margin-bottom: 20px;
    
    .name{
      margin-bottom: 10px;
    }
    .img-wrapper{
      
      img{
        border-radius: 50%;
      }
    }
  }
.fade-enter-active, .fade-leave-active {
  transition: opacity .5s
}
.fade-enter, .fade-leave-to{
  opacity: 0
}
}
</style>

<template>
  <div id="app">
    <div class="fb-login-button" data-max-rows="1" data-size="large" data-button-type="login_with" data-show-faces="false" data-auto-logout-link="true" data-use-continue-as="false" :data-scope="scope" onlogin="checkLoginState"></div>
    <br>
    <br>

    <transition name="fade">
    <div class="profile" v-if="authorized">
      <div class="name">{{ profile.name }}</div>
      <div class="img-wrapper">
        <img :src="profilePicture" alt="">
      </div>
    </div>
    </transition>

    <photos v-if="authorized"></photos>
    <!-- <router-view></router-view> -->
  </div>
</template>

