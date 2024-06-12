<template>
  <div class="index">
    <swiperVue />
    <div class="hot_books_container">
      <div class="hot_books_lists">
        <!-- 总榜 -->
        <el-card class="hot_books_card total_rank" shadow="always">
          <div slot="header" class="clearfix">
            <span>书籍热榜</span>
          </div>
          <el-table
            class="hot_books_table"
            :data="hotBooks"
            height="520"
            border
            style="width: 100%; font-size: 14px"
            v-loading="loading"
            element-loading-text="拼命加载中"
            element-loading-spinner="el-icon-loading"
            element-loading-background="rgba(0, 0, 0, 0.8)"
            stripe
          >
            <el-table-column
              label="序号"
              type="index"
              width="60"
              align="center"
            ></el-table-column>
            <el-table-column
              prop="bookName"
              label="书名"
              min-width="200"
            ></el-table-column>
          </el-table>
        </el-card>

        <!-- 小说榜单 -->
        <el-card class="hot_books_card novel_rank" shadow="always">
          <div slot="header" class="clearfix">
            <span>小说类热榜</span>
          </div>
          <el-table
            class="hot_books_table"
            :data="novelBooks"
            height="520"
            border
            style="width: 100%; font-size: 14px"
            v-loading="loading"
            element-loading-text="拼命加载中"
            element-loading-spinner="el-icon-loading"
            element-loading-background="rgba(0, 0, 0, 0.8)"
            stripe
          >
            <el-table-column
              label="序号"
              type="index"
              width="60"
              align="center"
            ></el-table-column>
            <el-table-column
              prop="bookName"
              label="书名"
              min-width="200"
            ></el-table-column>
          </el-table>
        </el-card>

        <!-- 计算机榜单 -->
        <el-card class="hot_books_card computer_rank" shadow="always">
          <div slot="header" class="clearfix">
            <span>计算机类热榜</span>
          </div>
          <el-table
            class="hot_books_table"
            :data="computerBooks"
            height="520"
            border
            style="width: 100%; font-size: 14px"
            v-loading="loading"
            element-loading-text="拼命加载中"
            element-loading-spinner="el-icon-loading"
            element-loading-background="rgba(0, 0, 0, 0.8)"
            stripe
          >
            <el-table-column
              label="序号"
              type="index"
              width="60"
              align="center"
            ></el-table-column>
            <el-table-column
              prop="bookName"
              label="书名"
              min-width="200"
            ></el-table-column>
          </el-table>
        </el-card>
      </div>
    </div>
  </div>
</template>

<script>
import swiperVue from "../../components/Swiper/swiper.vue";
export default {
  name: "index",
  components: {
    swiperVue,
  },
  data() {
    return {
      hotBooks: [],
      novelBooks: [],
      computerBooks: [],
      loading: true,
    };
  },
  created() {
    this.fetchHotBooks();
    this.fetchHotBooksByCategory("小说");
    this.fetchHotBooksByCategory("计算机");
  },
  methods: {
    async fetchHotBooks() {
      this.loading = true;
      try {
        const { data: res } = await this.$http.get("user/get_hot_books");
        if (res.status !== 200) {
          this.loading = false;
          return this.$message.error(res.msg);
        }
        this.hotBooks = res.data;
        this.loading = false;
      } catch (error) {
        this.loading = false;
        this.$message.error("获取热门书籍失败");
      }
    },
    async fetchHotBooksByCategory(category) {
      this.loading = true;
      try {
        const { data: res } = await this.$http.get(`user/get_hot_by_category?category=${category}`);
        if (res.status !== 200) {
          this.loading = false;
          return this.$message.error(res.msg);
        }
        if (category === '小说') {
          this.novelBooks = res.data;
        } else if (category === '计算机') {
          this.computerBooks = res.data;
        }
        this.loading = false;
      } catch (error) {
        this.loading = false;
        this.$message.error(`获取${category}热门书籍失败`);
      }
    },
  },
};
</script>

<style lang="css">
.hot_books_container {
  padding: 20px;
}
.hot_books_lists {
  display: flex;
  justify-content: space-between;
}
.hot_books_card {
  width: 32%;
  margin-bottom: 20px;
}
.total_rank {
  background-color: #E0F2E9;
}
.novel_rank {
  background-color: #fff0f0;
}
.computer_rank {
  background-color: #f0f0ff;
}
.hot_books_table {
  border-radius: 8px;
}
.el-table__body-wrapper {
  border-radius: 8px;
}
</style>
