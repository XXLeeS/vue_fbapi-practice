<script>
export default {
  name: 'photo-detail',
  props: ['id'],
  data(){
    return {
			likes: [],

    }
  },
  created(){
		this.fetchLikes();
  },
  methods: {
    fetchLikes(){
      FB.api(`/${this.id}/likes?limit=30`, response => {
					this.likes = this.likes.concat(response.data);
			});
    },
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
