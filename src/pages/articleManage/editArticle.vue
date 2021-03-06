<template>
  <div>
    <i-layout>
      <i-breadcrumb :t2="breadcrumbTitle">
        <Button class="fr vue-back-btn" @click="$router.go(-1)" shape="circle">返回</Button>
      </i-breadcrumb>
      <div class="vue-panel">
        <Form :model="formItem" ref="formItem" :rules="ruleValidate" :label-width="80">
          <FormItem label="文章标题:" prop="articleTitle">
            <Input clearable v-model="formItem.articleTitle" placeholder="请输入文章标题"></Input>
          </FormItem>
          <FormItem label="文章作者:" prop="userCode">
            <Select @on-change="getCategory" placeholder="请选择文章作者" clearable v-model="formItem.userCode">
              <Option :key="x.userCode" v-for="x in userList" :value="x.userCode">{{x.userName}}</Option>
            </Select>
          </FormItem>
          <FormItem label="文章分类:" prop="category">
            <Select not-found-text="该用户还没有文章分类" placeholder="请选择文章分类" clearable v-model="formItem.category" filterable multiple>
              <Option v-for="item in categoryList" :value="item._id" :key="item._id">{{ item.categoryName }}</Option>
            </Select>
          </FormItem>
          <FormItem label="文章内容:" prop="content">
            <editor class="editor" :value="formItem.content" v-model="formItem.content" :setting="editorSetting" @input="(content)=> content = content"></editor>
          </FormItem>
          <FormItem label="文章摘要:" prop="abstract">
            <Input v-model="formItem.abstract" type="textarea" :autosize="{minRows: 2,maxRows: 5}" placeholder="请输入文章摘要"></Input>
          </FormItem>
          <FormItem label="文章标签:" prop="tags">
            <Select clearable v-model="formItem.tags" filterable multiple>
              <Option v-for="item in cityList" :value="item.value" :key="item.value">{{ item.label }}</Option>
            </Select>
          </FormItem>
          <FormItem label="文章状态:" prop="status">
            <Select placeholder="请选择文章状态" clearable v-model="formItem.status">
              <Option value="100">发布</Option>
              <Option value="500">保存草稿</Option>
            </Select>
          </FormItem>
          <FormItem>
            <Button size="large" type="primary" class="vue-back-btn" @click="handleSubmit('formItem')" shape="circle">保存</Button>
            <Button size="large" type="ghost" class="vue-back-btn" @click="clear" shape="circle" style="margin-left: 8px">清空</Button>
          </FormItem>
        </Form>
      </div>

    </i-layout>
  </div>
</template>

<script>
import iLayout from "../../components/layout.vue";
import editor from "../../components/editor.vue";
import iBreadcrumb from "../../components/breadcrumb.vue";
import {
  saveArticle,
  getArticleList,
  updateArticle,
  getUserList,
  getCategoryList
} from "../../service/getData";
export default {
  name: "editArticle",
  components: {
    iLayout,
    editor,
    iBreadcrumb
  },
  created() {
    this.getUserList();
    if (this.$route.query && this.$route.query.id) {
      this.findArticleInfo(this.$route.query.id);
      this.breadcrumbTitle = "修改文章";
    } else {
      // 新增用户时提示一下
      this.breadcrumbTitle = "新增文章";
    }
  },
  data() {
    const validateCategory = (rule, value, callback) => {
      if (!this.formItem.userCode) {
        callback(new Error("请选择文章作者"));
      }
      if (value.length === 0) {
        callback(new Error("请选择文章分类"));
      } else {
        callback();
      }
    };
    const validateTags = (rule, value, callback) => {
      if (value.length === 0) {
        callback(new Error("请选择文章标签"));
      } else {
        callback();
      }
    };

    return {
      categoryList: [],
      breadcrumbTitle: "",
      userList: [],
      ruleValidate: {
        articleTitle: [
          {
            required: true,
            message: "请输入文章标题",
            trigger: "blur"
          }
        ],
        platType: [
          {
            required: true,
            message: "请选择所属平台",
            trigger: "change"
          }
        ],
        status: [
          {
            required: true,
            message: "请选择文章状态",
            trigger: "blur"
          }
        ],
        userCode: [
          {
            required: true,
            message: "请选择文章作者",
            trigger: "blur"
          }
        ],
        abstract: [
          {
            required: true,
            message: "请选择文章摘要",
            trigger: "blur"
          }
        ],
        tags: [
          {
            required: true,
            validator: validateTags,
            trigger: "change"
          }
        ],
        category: [
          {
            required: true,
            validator: validateCategory,
            trigger: "change"
          }
        ],
        content: [
          {
            required: true,
            message: "请输入文章内容",
            // validator: validateCategory,
            trigger: "change"
          }
        ]
      },
      cityList: [
        {
          value: "New York",
          label: "New York"
        },
        {
          value: "London",
          label: "London"
        },
        {
          value: "Sydney",
          label: "Sydney"
        },
        {
          value: "Ottawa",
          label: "Ottawa"
        },
        {
          value: "Paris",
          label: "Paris"
        },
        {
          value: "Canberra",
          label: "Canberra"
        }
      ],
      editorSetting: {
        height: 500
      },
      formItem: {
        category: [],
        tags: [],
        content: "",
        articleTitle: ""
      }
    };
  },
  methods: {
    async getCategory() {
      const res = await getCategoryList({
        params: {
          userCode: this.formItem.userCode
        }
      });
      if (res.respCode === "000000" && res.values) {
        this.categoryList = res.values;
      }
    },
    async findArticleInfo(id) {
      const res = await getArticleList({
        params: {
          _id: id
        },
        edit: true
      });
      if (res.respCode === "000000" && res.values) {
        this.formItem = res.values[0];
      }
    },
    // 获取用户列表
    async getUserList() {
      const res = await getUserList({
        params: {}
      });
      if (res && res.respCode === "000000") {
        if (res.values) {
          this.userList = res.values;
        } else {
          this.userList = [];
        }
      }
    },
    handleSubmit(name) {
      this.$refs[name].validate(valid => {
        if (valid) {
          this.save();
        } else {
          this.$Message.error("Fail!");
        }
      });
    },
    async save() {
      let res;
      if (this.$route.query && this.$route.query.id) {
        res = await updateArticle({
          ...this.formItem,
          id: this.$route.query.id
        });
      } else {
        res = await saveArticle(this.formItem);
      }
      if (res.respCode === "000000") {
        this.$Message.success(res.respMsg);
        setTimeout(() => {
          this.$router.push("/articleManage/articleList");
        }, 2000);
      } else {
        this.$Message.error(res.respMsg);
      }
    },
    clear() {
      this.content = "";
    }
  }
};
</script>

<style>

</style>
