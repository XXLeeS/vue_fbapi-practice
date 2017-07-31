<script>
export default {
  name: 'photo-detail',
  props: ['id'],
  data(){
    return {
      likes: [],
      likeIds: [],
      pageLikes: []

    }
  },
  created(){
		this.fetchLikes();
  },
  methods: {
    fetchLikes(){
      FB.api(`/${this.id}/likes?limit=300`, response => {
          this.likes = this.likes.concat(response.data);
          response.data.forEach(obj => {
            this.likeIds.push(obj.id);
          });

          this.fetchPageLikes(0);
			});
    },
    // you can get only the friends' data who install this fb app too. FUCK!
    fetchPageLikes(index){
      if(index >= this.likeIds.length){
        return;
      }
      // because FB api limit the max ids number to 50 
      const chunk = 50;
      let slicedIds = this.likeIds.slice(index, index + chunk);

      let str = slicedIds.join();
      FB.api(`likes?limit=1000&ids=${str}`, response => {
        for(let id in response){
          response[id].data.forEach(page => {
            // check if this page like exist in previous friends
            let existed = false;
            this.pageLikes.forEach((existPage, i) => {
              if(existPage.id === page.id){
                existPage.count += 1;
              }
            });

            if(!existed){
              let tmp = page;
              tmp.count = 1;
              this.pageLikes.push(tmp);
            }
          })
        }

        this.fetchPageLikes(index + chunk);
      })
    }
  },

}
</script>


<template>
  <div>
		<Row :gutter="16">
			<Col :span="6" v-for="like in likes" :key="like.id">
				{{ like.name }}
			</Col>
		</Row>
  </div>
</template>
