<template>
  <div class="blogdetail">
    <Header></Header>
    <div class="blog">
      <h2>{{blog.title}}</h2>
      <el-link icon="el-icon-edit" v-if="ownblog" class="linklist">
        <el-button  type="primary" round >
          <router-link :to="{name:'BlogEdit',params:{blogid:blog.id}}">编辑</router-link>
        </el-button>
      </el-link> &nbsp;
      <el-link icon="el-icon-delete" v-if="ownblog" class="linklist">
        <el-button type="danger" round @click="delblog">删除</el-button>
      </el-link>
      <el-divider></el-divider>

      <div class="markdown-body" v-html="blog.content"></div>
    </div>
  </div>
</template>

<script>
  //博客详情内容页面
  import 'github-markdown-css/github-markdown.css' // 然后添加样式markdown-body
  import Header from "../components/Header";

  export default {
    name: "BlogDetail",
    components: {
      Header
    },
    data() {
      return {
        blog: {
          id: '',
          title: "",
          description: "",
          content: ""
        },
        ownblog: false
      }
    },
    methods: {
      //删除blog
      delblog () {
        const blogId = this.$route.params.blogid
        const _this = this
        if (blogId) {
          this.$confirm('此操作将永久删除该文章, 是否继续?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }).then(() => {
            _this.$axios.post(`/blogdel/${blogId}`, null, {
              headers: {
                "Authorization": localStorage.getItem("token")
              }
            }).then(res => {
              this.$message({
                type: 'success',
                message: res.data.data
              });
              _this.$router.push("/blogs")
            })

          }).catch(() => {
            this.$message({
              type: 'info',
              message: '已取消删除'
            });
          });
        }
      }
    },
    created() {
      //获取动态路由的 blogid
      const blogId = this.$route.params.blogid
      const _this = this
      if (blogId) {
        this.$axios.get("/blog/" + blogId).then(res => {
          const blog = res.data.data
          _this.blog.id = blog.id
          _this.blog.title = blog.title
          _this.blog.description = blog.description

          //MardownIt 渲染
          var MardownIt = require("markdown-it")
          var md = new MardownIt();
          var result = md.render(blog.content)
          _this.blog.content = result

          //查看是否是登录人 是则可以编辑和删除
          _this.ownblog = (blog.userId === _this.$store.getters.getUser.id)
        })
      }
    },
    mounted(){
      this.$notify({
          title: '欢迎 ^=^ !',
          message: '阅读一下吧 ^-^ !',
          duration: 1500
      });
    }
  }
</script>

<style scoped>
  .blogdetail{
    width: 60%;
    margin: 0 auto;
    height: 1100px;
  }
  .mblog {
    margin-top: 10px;
    box-shadow: 0 2px 12px 0 rgba(252, 250, 250, 0.1);
    width: 100%;
    min-height: 700px;
    padding: 20px 15px;
    margin: 10px 10px;
  }
  .linklist {
    margin: 5px;
  }
  
/* github-markdown */
.markdown-body {
    box-sizing: border-box;
    min-width: 200px;
    margin: 0 auto;
    padding: 45px;
    border-radius: 1%;
}
 
@media (max-width: 767px) {
    .markdown-body {
        padding: 15px;
    }
}
</style>