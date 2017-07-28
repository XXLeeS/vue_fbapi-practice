<script>
export default {
  name: 'photos',
  data(){
      return{
        photos: [],
        pagingNext: '',
        expanded: [],
        sizeIndex: 5,    // specific the size of returned images by arr index
      }
  },
  created(){
    this.getPhotos();
  },
  methods: {
    getPhotos(){
        FB.api('/me/photos?limit=24&type=uploaded&fields=images,link,likes.summary(true).limit(0),comments.limit(100){comment_count}', response => {
            this.setResponse(response);
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
            this.setResponse(json);
        })
        .catch(error => {
            console.error(error);
        })
    },
    setResponse(res){
        let arr = new Array(res.data.length).fill(false);
        this.expanded = this.expanded.concat(arr);
        this.photos = this.photos.concat(res.data);
        this.pagingNext = res.paging.next;
    },
    sumComment(commentsArr){
        let total = commentsArr.reduce((sum, obj) => {
            return sum + obj.comment_count;
        }, 0);
        return total;
    },

    expandDetail(index){
        if(this.expanded[index]){
            this.closeDetail(index);
        }
        else{
            this.expanded.fill(false);
            this.$set(this.expanded, index, true);
        }
    },
    closeDetail(index){
        this.$set(this.expanded, index, false);
    },
    getOffset(i){
        return `${i * -100}%`;
    },

    afterEnterStretch(el){
        el.classList.add('stretched');
    },
    beforeLeaveStretch(el){
        el.classList.remove('stretched');
    }
  }

}
</script>

<style lang="scss">
#photos{
    .scale-enter-active, .scale-leave-active{
        transition: all .5s;
    }
    .scale-enter, .scale-leave-to{
        opacity: 0;
        transform: scale(.5);
    }

    $stretch_height: 300px;
    .stretch-enter-active, .stretch-leave-active{
        transition: all .3s;
    }
    .stretch-enter, .stretch-leave-to{
        height: 0;
    }
    .stretch-enter-to, .stretch-leave{
        height: $stretch_height;
    }

    .photo__brief{
        position: relative;
        cursor: pointer;

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
    }

    .arrow-up{
        margin: 3px auto 0;
        width: 0;
        height: 0;
        border-left: 10px solid transparent;
        border-right: 10px solid transparent;
        border-bottom: 10px solid #333;
    }
    .photo__detail{
        position: relative;
        width: 400%;
        // height: 200px;
        background-color: #333;
        color: #fff;

        &.stretched{
            height: $stretch_height;
        }

        .close-btn{
            position: absolute;
            right: 15px;
            top: 15px;
            cursor: pointer;
            color: #999;
            
            &:hover{
                color: #fff;
            }
        }
    }

}

</style>


<template>
    <div id="photos">
        <!-- <Row :gutter="16"> -->
        <transition-group name="scale" tag="div" class="ivu-row ivu-row-flex ivu-row-flex-top"> 
            <div class="ivu-col ivu-col-span-6 margin-bottom"
                v-for="(photo, index) in photos" :key="photo.id">
                <!-- <a :href="photo.link" target="_blank"> -->
                    <div class="photo__brief" @click="expandDetail(index)" style="padding: 0 8px;">
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

                        <transition :duration="300"> 
                        <div v-if="expanded[index]" class="arrow-up"></div>
                        </transition> 
                    </div>
                <!-- </a> -->
                <transition name="stretch" @after-enter="afterEnterStretch" @before-leave="beforeLeaveStretch">
                <div v-if="expanded[index]" class="photo__detail" :style="`margin-left: ${getOffset(index%4)};`">
                    <div class="close-btn" @click="closeDetail(index)">
                        <Icon type="close"></Icon>
                    </div>
                    <div>
                        hey
                        <br>
                        hey
                        <br>
                    </div>
                </div>
                </transition>
            </div>                
        </transition-group> 
        <!-- </Row> -->
        <Row class="margin-bottom">
            <Col :span="6" offset="9">
                <Button @click="getPaging" size="large" long>Load More</Button>
            </Col>
        </Row>
    </div>
</template>

