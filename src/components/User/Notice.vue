<template>
  <div class="notice_container">
    <div class="header">
      <div class="scroll-text" ref="scrollText">
        <i class="el-icon-s-opportunity"></i> {{ text[currentIndex]}}
        <i class="el-icon-s-opportunity"></i>
      </div>
    </div>
    <div class="banner">
      <div class="banner_header"><p>近期公告</p></div>
      <div class="banner_main"
      v-loading="loading"
        element-loading-text="拼命加载中"
    element-loading-spinner="el-icon-loading"
    element-loading-background="rgba(0, 0, 0, 0.8)">
        <div class="banner_main_item" v-for="item in noticeList" :key="item.noticeId">
          <div class="banner_main_item_header">    <p> {{ item.noticeTitle }}     {{ item.createTime }}</p></div>
       
          <div class="banner_main_item_main">
                <p>{{ item.noticeContent }}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>

export default {
  data() {
    return {
      text: [
        "世上只有想不通的人，没有走不到的路，只要你肯借我一本书看看。",
  "每天进步一点点，换来的是一生的快乐，每天阅读一点点，走出的是一片天地。",  
      "悄悄地读书，说话的不要",
      "你知道的，本图书馆向来以高人气著称",
      "书山有路勤为径，学海无涯苦作舟，来呀，快活呀，反正有大把时光。",


      ],
        currentIndex: 0, // 当前显示的文本索引
      noticeList:[
        {
          noticeId:0,
          noticeAdminId:Number,
          noticeTitle:"",
          noticeContent:"",
          createTime:"",
          updateTime:""
        }
      ],
      loading:true
    };
  },
  methods: {
    async getRuleList(){
      this.loading = true;
      const {data:res} = await this.$http.get('user/get_noticelist')
      if(res.status!== 200){
        this.loading = false;
        return this.$message.error(res.msg)
      }
      this.$message.success({
        message:res.msg,
        duration:1000
      })
    this.noticeList = res.data.reverse(); // 逆序排列公告列表
      this.loading = false;
    },
     changeText() {
      this.currentIndex = (this.currentIndex + 1) % this.text.length;
    },
    
  },
  mounted() {
    const containerWidth = this.$refs.scrollText.offsetWidth;
    const textWidth = this.$refs.scrollText.scrollWidth;

    // If the text is longer than the container, start the animation
    if (textWidth > containerWidth) {
      this.$refs.scrollText.style.animation = "scroll 10s linear infinite";
    };
     // 每隔一段时间切换一次文本
    setInterval(this.changeText, 10000); // 5000毫秒为切换间隔，可以根据需求调整
  },
  created(){
    this.getRuleList();
  }
};
</script>

<style lang="less" scoped>
.notice_container {
  overflow: hidden;
}
.header {
  width: 100%;
  height: 50px;
  background-color: rgb(70, 130, 180);
  border-radius: 15px;
  // box-shadow: 0px 0px 5px 5px rgb(66, 142, 5);
  color: white;
  text-align: center;
  line-height: 50px;
  font-size: 24px;
}
.scroll-text {
  white-space: nowrap;
  animation: scroll 10s linear infinite;
}

@keyframes scroll {
  from {
    transform: translateX(100%);
  }
  to {
    transform: translateX(-100%);
  }
}
.banner {
  width: 100%;
  height: 100%;
  // background-color: pink;
  margin-top:30px;
}
.banner_header {
  width: 100%;
  height: 80px;
  // background-color: brown;
 
  p {
    color:black;
    font-size: 30px;
    text-align: center;
    line-height: 80px;
  }
}
.banner_main{
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  color:skyblue;

}

.banner_main_item:nth-child(n+2) {
  margin-top: 30px;
}
.banner_main_item:nth-child(n+2){
  background-color: #D1EEEE;
}
.banner_main_item:nth-child(1){
  background-color: pink;
}

.banner_main_item {
  width: 80%;
  height: 120px;
  // background-color: aqua;
  .banner_main_item_header {
    width: 100%;
    height: 50px;
    // background-color: pink;
    border:1px solid skyblue;
    box-sizing: border-box;
    p {
      color:rgb(175, 129, 143);
      text-align: center;
      line-height: 50px;
      font-size: 16px;
    }
  }
  .banner_main_item_main{
    width: 100%;
    height: 70px;
    background-color: white;
    border:1px solid skyblue;
    box-sizing: border-box;
    text-align: center;
    p {
      line-height: 70px;
    }
  }
}
</style>