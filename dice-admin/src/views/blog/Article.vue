<template>
  <div class="app-container">
    <el-form
      ref="articleForm"
      label-position="top"
      :rules="rules"
      :model="article"
    >
      <el-row :gutter="30">
        <el-col>
          <el-form-item prop="title">
            <el-input
              v-model="article.title"
              placeholder="请输入文章标题"
            />
          </el-form-item>
          <el-form-item prop="content">
            <markdown-editor
              v-model="article.content"
              :on-save="handleSaveDraft"
            />
            <!-- 键修饰符，键别名 -->
          </el-form-item>
        </el-col>
      </el-row>
    </el-form>
    <el-dialog
      :visible.sync="mediaDialog"
      top="10vh"
      :fullscreen="isMobile"
      append-to-body
      center
      width="80%"
    >
      <upload :after-upload="afterUpload" />
      <div class="media-list">
        <el-row>
          <el-col
            v-for="media in mediaDialogData.mediaDatas"
            :key="media.id"
            style="padding: 6px"
            :xs="24"
            :sm="12"
            :md="12"
            :lg="6"
            :xl="4"
          >
            <media-item
              :media="media"
              :after-delete="afterDeleteMedia"
            />
          </el-col>
        </el-row>
        <pagination v-show="mediaDialogData.total>0" :total="mediaDialogData.total" :page.sync="mediaDialogData.pageNum" :limit.sync="mediaDialogData.pageSize" @pagination="showMediaDialog" />
      </div>
    </el-dialog>
    <el-drawer
      :visible.sync="postSettingVisible"
      :with-header="false"
      size="20%"
    >
      <el-form
        label-position="top"
        :rules="rules"
        :model="article"
        style="margin-left: 16px;margin-right: 16px"
      >
        <el-form-item label="标签">
          <el-select
            v-model="selectTags"
            multiple
            filterable
            placeholder="请选择文章标签"
            style="width: 100%"
          >
            <el-option
              v-for="tag in tags"
              :key="tag.value"
              :label="tag.label"
              :value="tag.value"
            />
          </el-select>
        </el-form-item>
        <el-form-item label="分类">
          <el-select
            v-model="article.category"
            style="width: 100%"
            filterable
            placeholder="请选择文章分类"
          >
            <el-option
              v-for="category in categories"
              :key="category.value"
              :label="category.label"
              :value="category.value"
            />
          </el-select>
        </el-form-item>
        <el-form-item label="状态">
          <el-switch
            v-model="article.status"
            :active-value="postStatus.PUBLISHED.value"
            :inactive-value="postStatus.DRAFT.value"
            active-text="公开"
            inactive-text="隐藏"
          />
        </el-form-item>
        <el-form-item>
          <el-switch
            v-model="article.priority"
            active-value="1"
            inactive-value="0"
            active-text="置顶"
            inactive-text="普通"
          >
          </el-switch>
        </el-form-item>
        <el-form-item>
          <el-switch
            v-model="article.allowComment"
            active-text="开启评论"
            inactive-text="关闭"
          >
          </el-switch>
        </el-form-item>
        <el-form-item label="创建日期">
          <el-date-picker
            v-model="article.createTime"
            style="width: 100%"
            type="datetime"
            placeholder="创建日期"
            size="small"
            :editable="flagFalse"
            value-format="timestamp"
          />
        </el-form-item>
        <el-form-item label="修改日期">
          <el-date-picker
            v-model="article.updateTime"
            style="width: 100%"
            type="datetime"
            placeholder="修改日期"
            size="small"
            :editable="flagFalse"
            value-format="timestamp"
          />
        </el-form-item>
        <el-form-item>
          <el-button
            size="small"
            @click="showMediaDialog"
          >媒体库
          </el-button>
        </el-form-item>
      </el-form>
      <div class="bottom-control">
        <el-button
          type="primary"
          size="small"
          @click="handleSaveDraft"
        >保存草稿
        </el-button>
        <el-button
          type="primary"
          size="small"
          @click="handlePublishClick"
        >发布
        </el-button>
        <el-button
          v-if="article.id !== ''"
          type="primary"
          size="small"
        >
          <a
            :href="
              this.$serverConfig.frontUrl +
                'article/' +
                article.id "
            target="_blank"
            style="color: #FFFFFF;"
          >查看</a>
        </el-button>
      </div>
    </el-drawer>
    <footer-tool-bar>
      <el-button
        type="primary"
        size="small"
        @click="handleSaveDraft"
      >保存草稿</el-button>
      <!--      <a-button-->
      <!--        style="margin-left: 8px;"-->
      <!--        @click="handlePreview"-->
      <!--      >预览</a-button>-->
      <el-button
        type="primary"
        size="small"
        style="margin-left: 8px;"
        @click="handleShowPostSetting"
      >发布</el-button>
      <!--      <el-button-->
      <!--        type="dashed"-->
      <!--        size="small"-->
      <!--        style="margin-left: 8px;"-->
      <!--        @click="()=>this.attachmentDrawerVisible = true"-->
      <!--      >附件库</el-button>-->
    </footer-tool-bar>
  </div>
</template>

<script>
import MarkdownEditor from '../../components/MarkdownEditor/MarkdownEditor'
import Upload from '../../components/Upload/Upload'
import MediaItem from '../../components/Upload/MediaItem'
import { pageMedia } from '@/api/media'
import Pagination from '@/components/Pagination' // secondary package based on el-pagination
import FooterToolBar from '@/components/FooterToolbar'
import * as blogApi from '@/api/blog'

export default {
  components: {
    MarkdownEditor,
    MediaItem,
    Upload,
    Pagination,
    FooterToolBar
  },
  data: function() {
    return {
      postStatus: blogApi.postStatus(),
      isMobile: false,
      mediaDialog: false,
      postSettingVisible: false,
      article: {
        id: '',
        title: '',
        tags: '',
        category: '',
        content: '',
        status: '',
        createTime: '',
        updateTime: '',
        priority: 0,
        allowComment: true
      },
      rules: {
        title: [
          { required: true, message: '文章标题必须输入', trigger: 'blur' }
        ],
        content: [
          { required: true, message: '文章内容不能为空', trigger: 'blur' }
        ]
      },
      selectTags: [],
      tags: [],
      categories: [],
      flagFalse: false,
      mediaDialogData: {
        mediaDatas: [],
        total: 0,
        pageSize: this.$static.DEFAULT_PAGE_SIZE,
        pageNum: 1
      }
    }
  },
  watch: {
    // 监听route刷新绑定的article数据
    $route() {
      this.getArticle()
    }
  },
  mounted() {
    this.init()
  },
  methods: {
    getArticle() {
      const id = this.$route.params.id
      // 如果有id则表示编辑文章,获取文章信息
      if (id) {
        blogApi.getArticle(id).then(response => {
          this.initArticle(response.data)
        })
      } else {
        // 如果没有id则表示新增文章,不用清空文章信息
        const data = {
          id: '',
          title: '',
          tags: '',
          category: '',
          content: '',
          status: blogApi.postStatus().PUBLISHED.value,
          createTime: Date.now(),
          updateTime: Date.now()
        }
        this.initArticle(data)
      }
    },
    initArticle(data) {
      this.article.id = data.id
      this.article.title = data.title
      this.article.tags = data.tags
      this.article.category = data.category
      this.article.content = data.content
      this.article.status = data.status
      this.article.createTime = new Date(data.createTime).getTime()
      this.article.updateTime = Date.now()
      this.selectTags = this.$util.stringToTags(data.tags)
    },
    getTags() {
      blogApi.getAllTags().then(response => {
        for (const key in response.data) {
          const tag = {
            value: response.data[key].name,
            label: response.data[key].name
          }
          this.tags.push(tag)
        }
      })
    },
    getCategories() {
      blogApi.getAllCategories().then(response => {
        for (const key in response.data) {
          const category = {
            value: response.data[key].name,
            label: response.data[key].name
          }
          this.categories.push(category)
        }
      })
    },
    submitArticle(formName, success) {
      this.$refs[formName].validate(valid => {
        if (valid) {
          const params = this.article
          params.tags = this.$util.tagsToString(this.selectTags)
          blogApi.saveArticle(params).then(response => {
            success(response.data)
          })
        }
      })
    },
    handlePublishClick() {
      const _this = this
      this.article.status = blogApi.postStatus().PUBLISHED.value
      this.submitArticle('articleForm', function() {
        _this.$util.message.success('发布文章成功!')
        _this.$router.push('/blog/article')
      })
    },
    init() {
      this.getArticle()
      this.getTags()
      this.getCategories()
    },
    showMediaDialog() {
      this.isMobile = document.body.clientWidth < 768
      this.mediaDialog = true
      pageMedia(this.mediaDialogData.pageSize, this.mediaDialogData.pageNum).then(response => {
        this.mediaDialogData.mediaDatas = response.data.list
        this.mediaDialogData.total = response.data.total
        this.mediaDialogData.pageSize = response.data.pageSize
        this.mediaDialogData.pageNum = response.data.pageNum
        for (const media of this.mediaDialogData.mediaDatas) {
          if (media.thumbUrl && media.thumbUrl !== '') {
            media.showUrl = this.$util.getServerMediaUrl(media.thumbUrl)
          } else {
            media.showUrl = this.$util.getServerMediaUrl(media.url)
          }
        }
      })
    },
    afterDeleteMedia(data) {
      if (data.success) {
        this.showMediaDialog(1)
      }
    },
    afterUpload(response) {
      if (response.success) {
        pageMedia(this.mediaDialogData.pageSize, this.mediaDialogData.pageNum)
          .then(response => {
            this.mediaDialogData.mediaDatas = response.data.list
            this.mediaDialogData.total = response.data.total
            this.mediaDialogData.pageSize = response.data.pageSize
            this.mediaDialogData.pageNum = response.data.pageNum
            for (const media of this.mediaDialogData.mediaDatas) {
              if (media.thumbUrl && media.thumbUrl !== '') {
                media.showUrl = this.$util.getServerMediaUrl(media.thumbUrl)
              } else {
                media.showUrl = this.$util.getServerMediaUrl(media.url)
              }
            }
          })
      }
    },
    handleSaveDraft() {
      const _this = this
      this.article.status = blogApi.postStatus().DRAFT.value
      this.submitArticle('articleForm', function(data) {
        _this.$util.message.success('草稿保存成功!')
        _this.$route.params.id = data
        _this.getArticle()
      })
    },
    handleShowPostSetting() {
      this.postSettingVisible = true
    }
  }
}
</script>
