<script>
export default {
  name: 'photos',
  data(){
      return{
        photos: [],
        pagingNext: '',
        sizeIndex: 5,    // specific the size of returned images by arr index
      }
  },
  created(){
    this.getPhotos();
  },
  methods: {
    getPhotos(){
        FB.api('/me/photos?limit=24&type=uploaded&fields=images,link,likes.summary(true).limit(0),comments.limit(100){comment_count}', response => {
            this.photos = this.photos.concat(response.data);
            this.pagingNext = response.paging.next;
        });
    },
    getPaging(){
        fetch(this.pagingNext)
        .then(res => {
            if(!res.ok){
                console.log(res.status);
            }

            return res.json();
        })
        .then(json => {
            this.photos = this.photos.concat(json.data);
            this.pagingNext = json.paging.next;
        })
        .catch(error => {
            console.error(error);
        })
    },
    // getComments(ids){
    //     let str = ids.join();
    //     FB.api(`/?ids=${ids}`, response => {
    //         this.$set(this, 'photos', response.data);
    //     });
    // },
    sumComment(commentsArr){
        let total = commentsArr.reduce((sum, obj) => {
            return sum + obj.comment_count;
        }, 0);
        return total;
    }
  }

}
</script>

<style lang="scss">
#photos{
    .image{
        width: 100%;
        padding-top: 100%;
        background-size: cover;
        background-position: center center;
    }
    .data{
        text-align: left;
        font-size: 18px;
        padding: 10px;
        color: #333;
    }
    .scale-enter-active, .scale-leave-active{
        transition: all .5s;
    }
    .scale-enter, .scale-leave-to{
        opacity: 0;
        transform: scale(.5);
    }
}

</style>


<template>
    <div id="photos">
        <!-- <Row :gutter="16"> -->
        <transition-group name="scale" tag="div" class="ivu-row"> 
            <Col v-for="photo in photos" :key="photo.id" span="6" class="margin-bottom" style="padding: 0 8px;">
                <a :href="photo.link" target="_blank">
                    <Card :padding="0">
                        <div class="image" :style="`background-image: url(${photo.images[sizeIndex].source});`"></div>
                        <div class="data">
                            <div><Icon type="thumbsup"></Icon> {{ photo.likes.summary.total_count }}</div>
                            <div v-if="photo.comments">
                            <!-- 不加 v-if 就會爆 -->
                                <div>直接留言數: {{ photo.comments.data.length }}</div>
                                <div>總留言數: {{ photo.comments.data.length + sumComment(photo.comments.data) }}</div>
                            </div>
                            <div v-else>
                                <div>直接留言數: 0</div>
                                <div>總留言數: 0</div>
                            </div>
                        </div>
                    </Card>
                </a>
            </Col>                
        </transition-group> 
        <!-- </Row> -->
        <Row class="margin-bottom">
            <Col :span="6" offset="9">
                <Button @click="getPaging" size="large" long>Load More</Button>
            </Col>
        </Row>
    </div>
</template>

