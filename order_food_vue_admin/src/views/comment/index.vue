<template>
  <el-card class="main-card">
    <div class="title">{{ this.$route.meta.menuName }}</div>
    <!-- 表格操作 -->
    <div class="operation-container" style="margin-top: 24px">
      <el-button
          type="danger"
          size="small"
          icon="el-icon-delete"
          :disabled="commentIdList.length == 0"
          @click="remove = true"
      >
        批量删除
      </el-button>
      <!-- 数据筛选 -->
      <div style="margin-left:auto;margin-top: 3px">
        <!--        <el-select-->
        <!--          v-model="type"-->
        <!--          placeholder="请选择来源"-->
        <!--          size="small"-->
        <!--          style="margin-right:1rem"-->
        <!--        >-->
        <!--          <el-option-->
        <!--            v-for="item in options"-->
        <!--            :key="item.value"-->
        <!--            :label="item.label"-->
        <!--            :value="item.value"-->
        <!--          />-->
        <!--        </el-select>-->
        <el-input
            v-model="keywords"
            prefix-icon="el-icon-search"
            size="small"
            placeholder="请输入评论人昵称"
            style="width:200px"
            @keyup.enter.native="searchComments"
        />
        <el-button
            type="primary"
            size="small"
            icon="el-icon-search"
            style="margin-left:1rem"
            @click="searchComments"
        >
          搜索
        </el-button>
      </div>
    </div>
    <!-- 表格展示 -->
    <el-table
        border
        :data="commentList"
        @selection-change="selectionChange"
        v-loading="loading"
    >
      <!-- 表格列 -->
      <el-table-column type="selection" width="55" />
      <el-table-column prop="avatar" label="头像" align="center" width="120">
        <template slot-scope="scope">
          <img :src="scope.row.avatar" width="40" height="40" />
        </template>
      </el-table-column>
      <!-- 评论人昵称 -->
      <el-table-column
          prop="nickname"
          label="评论人"
          align="center"
          width="120"
      />
      <!-- 回复人昵称 -->
      <el-table-column
          prop="replyNickname"
          label="回复人"
          align="center"
          width="120"
      >
        <template slot-scope="scope">
          <span v-if="scope.row.replyNickname">
            {{ scope.row.replyNickname }}
          </span>
          <span v-else>无</span>
        </template>
      </el-table-column>
      <!-- 评论食物名称 -->
      <el-table-column prop="articleTitle" label="食物名称" align="center">
        <template slot-scope="scope">
          <span v-if="scope.row.title">
            {{ scope.row.title }}
          </span>
          <span v-else>无</span>
        </template>
      </el-table-column>
      <!-- 评论内容 -->
      <el-table-column prop="commentContent" label="评论内容" align="center">
        <template slot-scope="scope">
          <span v-html="scope.row.content" class="comment-content" />
        </template>
      </el-table-column>
      <!-- 评论时间 -->
      <el-table-column
          prop="createTime"
          label="评论时间"
          width="150"
          align="center"
      >
        <template slot-scope="scope">
          <i class="el-icon-time" style="margin-right:5px" />
          {{ scope.row.createTime | date }}
        </template>
      </el-table-column>
      <!--      &lt;!&ndash; 状态 &ndash;&gt;-->
      <!--      <el-table-column prop="isReview" label="状态" width="80" align="center">-->
      <!--        <template slot-scope="scope">-->
      <!--          <el-tag v-if="scope.row.isReview == 0" type="warning">审核中</el-tag>-->
      <!--          <el-tag v-if="scope.row.isReview == 1" type="success">正常</el-tag>-->
      <!--        </template>-->
      <!--      </el-table-column>-->
      <!--      &lt;!&ndash; 来源 &ndash;&gt;-->
      <!--      <el-table-column label="来源" align="center" width="100">-->
      <!--        <template slot-scope="scope">-->
      <!--          <el-tag v-if="scope.row.articleTitle">食物</el-tag>-->
      <!--          <el-tag v-else type="warning">友链</el-tag>-->
      <!--        </template>-->
      <!--      </el-table-column>-->
      <!-- 列操作 -->
      <el-table-column label="操作" width="160" align="center">
        <template slot-scope="scope">
          <el-button
              v-if="scope.row.isReview == 0"
              size="mini"
              type="success"
              slot="reference"
              @click="updateCommentReview(scope.row.id)"
          >
            通过
          </el-button>
          <el-popconfirm
              style="margin-left:10px"
              title="确定删除吗？"
              @confirm="deleteComments(scope.row.id)"
          >
            <el-button size="mini" type="danger" slot="reference">
              删除
            </el-button>
          </el-popconfirm>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页 -->
    <el-pagination
        class="pagination-container"
        background
        @size-change="sizeChange"
        @current-change="currentChange"
        :current-page="current"
        :page-size="size"
        :total="count"
        :page-sizes="[10, 20]"
        layout="total, sizes, prev, pager, next, jumper"
    />
    <!-- 批量彻底删除对话框 -->
    <el-dialog :visible.sync="remove" width="30%">
      <div class="dialog-title-container" slot="title">
        <i class="el-icon-warning" style="color:#ff9900" />提示
      </div>
      <div style="font-size:1rem">是否彻底删除选中项？</div>
      <div slot="footer">
        <el-button @click="remove = false">取 消</el-button>
        <el-button type="primary" @click="deleteComments(null)">
          确 定
        </el-button>
      </div>
    </el-dialog>
  </el-card>
</template>

<script>
import {deleteCommentList, getCommentPage} from "@/api/comment";

export default {
  name:"MyComment",
  created() {
    this.listComments();
  },
  data: function() {
    return {
      loading: true,
      remove: false,
      options: [
        {
          value: 1,
          label: "食物"
        },
        {
          value: 2,
          label: "友链"
        }
      ],
      commentList: [],
      commentIdList: [],
      type: null,
      keywords: null,
      isReview: null,
      current: 1,
      size: 10,
      count: 0
    };
  },
  methods: {
    selectionChange(commentList) {
      this.commentIdList = [];
      commentList.forEach(item => {
        this.commentIdList.push(item.id);
      });
    },
    searchComments() {
      this.current = 1;
      this.listComments();
    },
    sizeChange(size) {
      this.size = size;
      this.listComments();
    },
    currentChange(current) {
      this.current = current;
      this.listComments();
    },
    changeReview(review) {
      this.isReview = review;
    },
    updateCommentReview(id) {
      let param = {};
      if (id != null) {
        param.idList = [id];
      } else {
        param.idList = this.commentIdList;
      }
      param.isReview = 1;
      this.axios.put("/api/admin/comments/review", param).then(({ data }) => {
        if (data.flag) {
          this.$notify.success({
            title: "成功",
            message: data.message
          });
          this.listComments();
        } else {
          this.$notify.error({
            title: "失败",
            message: data.message
          });
        }
      });
    },
    async deleteComments(id) {
      let param = [];
      if (id == null) {
        param = this.commentIdList;
      } else {
        param =  [id];
      }
      const res = await deleteCommentList(param);
      if (res.code) {
        this.$notify.success({
          title: "成功",
          message: res.msg
        });
        await this.listComments();
      } else {
        this.$notify.error({
          title: "失败",
          message: res.msg
        });
      }
      this.remove = false;
    },
    async listComments() {
      const param = {
        currentPage: this.current,
        pageSize: this.size,
        queryString: this.keywords
      };
      const res = await getCommentPage(param);
      this.commentList = res.data.records;
      this.count = res.data.total;
      this.loading = false;
    }
  },
  watch: {
    isReview() {
      this.current = 1;
      this.listComments();
    },
    type() {
      this.current = 1;
      this.listComments();
    }
  }
};
</script>

<style scoped>
.comment-content {
  display: inline-block;
}
.operation-container {
  margin-top: 1.5rem;
}
.review-menu {
  font-size: 14px;
  margin-top: 40px;
  color: #999;
}
.review-menu span {
  margin-right: 24px;
}
.review {
  cursor: pointer;
}
.active-review {
  cursor: pointer;
  color: #333;
  font-weight: bold;
}
</style>
