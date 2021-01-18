<template>
  <div class="announcement view">
    <Panel :title="$t('m.Problem_Classify')">
      <div class="list">
        <el-table
          v-loading="loading"
          element-loading-text="loading"
          ref="table"
          :data="announcementList"
          style="width: 100%">
          <el-table-column
            width="100"
            prop="id"
            label="ID">
          </el-table-column>
          <el-table-column
            label="Name">
              <template slot-scope="scope">
              <span v-if="scope.row.parent">---------{{scope.row.name}}</span>
              <span v-if="!scope.row.parent">{{scope.row.name}}</span>
            </template>
          </el-table-column>
          <el-table-column
            fixed="right"
            label="Option"
            width="400">
            <div slot-scope="scope">
              <icon-btn name="Edit" icon="edit" @click.native="openClassifyDialog(scope.row.id)" v-if="!scope.row.parent"></icon-btn>
              <icon-btn name="Edit" icon="edit" @click.native="openSubclassDialog(scope.row.id, scope.row.parent)" v-if="scope.row.parent"></icon-btn>
              <icon-btn name="Delete" icon="trash" @click.native="deleteClassify(scope.row.id)" v-if="!scope.row.parent"></icon-btn>
              <icon-btn name="Delete" icon="trash" @click.native="deleteSubclass(scope.row.id)" v-if="scope.row.parent"></icon-btn>
              <el-button type="primary" size="small" @click="openSubclassDialog(scope.row.id, scope.row.parent)" icon="el-icon-plus" v-if="!scope.row.parent">子类</el-button>
            </div>
          </el-table-column>
        </el-table>
        <div class="panel-options">
          <el-button type="primary" size="small" @click="openClassifyDialog(null)" icon="el-icon-plus">Create</el-button>
          <el-pagination
            v-if="!contestID"
            class="page"
            layout="prev, pager, next"
            @current-change="currentChange"
            :page-size="pageSize"
            :total="total">
          </el-pagination>
        </div>
      </div>
    </Panel>
    <!--对话框-->
    <el-dialog :title="announcementDialogTitle" :visible.sync="showEditAnnouncementDialog"
               @open="onOpenEditDialog" :close-on-click-modal="false">
      <el-form label-position="top">
        <el-form-item :label="$t('m.Classify')" required>
          <el-input
            v-model="announcement.title"
            :placeholder="$t('m.Classify')" class="title-input">
          </el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
          <cancel @click.native="showEditAnnouncementDialog = false"></cancel>
          <save type="primary" @click.native="submitAnnouncement"></save>
        </span>
    </el-dialog>
    <!--子类对话框-->
    <el-dialog :title="announcementDialogTitle" :visible.sync="showEditSubclassDialog"
               @open="onOpenEditDialog" :close-on-click-modal="false">
      <el-form label-position="top">
        <el-form-item :label="$t('m.Subclass')" required>
          <el-input
            v-model="announcement.title"
            :placeholder="$t('m.Subclass')" class="title-input">
          </el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
          <cancel @click.native="showEditSubclassDialog = false"></cancel>
          <save type="primary" @click.native="submitSubclass"></save>
        </span>
    </el-dialog>
  </div>
</template>

<script>
  import Simditor from '../../components/Simditor.vue'
  import api from '../../api.js'

  export default {
    name: 'Announcement',
    components: {
      Simditor
    },
    data () {
      return {
        contestID: '',
        // 显示编辑公告对话框
        showEditAnnouncementDialog: false,
        showEditSubclassDialog: false,
        // 公告列表
        announcementList: [],
        subclassList: [],
        // 一页显示的公告数
        pageSize: 15,
        // 总公告数
        total: 0,
        // 当前公告id
        currentAnnouncementId: null,
        subclassId: null,
        mode: 'create',
        // 公告 (new | edit) model
        announcement: {
          title: '',
          visible: true,
          content: ''
        },
        // 对话框标题
        announcementDialogTitle: 'Edit Announcement',
        // 是否显示loading
        loading: true,
        // 当前页码
        currentPage: 0
      }
    },
    mounted () {
      this.init()
    },
    methods: {
      init () {
        this.contestID = this.$route.params.contestId
        if (this.contestID) {
          this.getContestAnnouncementList()
        } else {
          this.getClassifyList(1)
        }
        this.getSubclassList()
      },
      // 切换页码回调
      currentChange (page) {
        this.currentPage = page
        this.getClassifyList(page)
      },
      getClassifyList (page) {
        this.loading = true
        api.getClassifyList((page - 1) * this.pageSize, this.pageSize).then(res => {
          this.loading = false
          this.total = res.data.data.total
          this.announcementList = res.data.data.results
        }, res => {
          this.loading = false
        })
      },
      getSubclassList () {
        this.loading = true
        api.getSubclassList().then(res => {
          this.loading = false
          this.subclassList = res.data.data.results
          // console.log(this.subclassList)
          // this.announcementList = this.announcementList.concat(this.subclassList)
          // console.log('wa', this.announcementList)
          let a = this.subclassList
          let b = this.announcementList
          let aLang = this.subclassList.length
          let bLang = this.announcementList.length
          // console.log(111, a)
          for (let i = 0; i < aLang; i++) {
            // console.log(a[i].parent)
            for (let j = 0; j < bLang + aLang; j++) {
              // console.log('j', j)
              let c = String(b[j].id)
              if (a[i].parent === c) {
                // console.log(i)
               // b = b.slice(, 1)
                b.splice(j + 1, 0, a[i])
               // console.log(b)
                break
              }
            }
          }
        }, res => {
          this.loading = false
        })
      },
      getContestAnnouncementList () {
      },
      // 打开编辑对话框的回调
      onOpenEditDialog () {
        // todo 优化
        // 暂时解决 文本编辑器显示异常bug
        setTimeout(() => {
          if (document.createEvent) {
            let event = document.createEvent('HTMLEvents')
            event.initEvent('resize', true, true)
            window.dispatchEvent(event)
          } else if (document.createEventObject) {
            window.fireEvent('onresize')
          }
        }, 0)
      },
      // 提交编辑
      // 默认传入MouseEvent
      submitAnnouncement (data = undefined) {
        let funcName = ''
        if (!data.title) {
          data = {
            id: this.currentAnnouncementId,
            name: this.announcement.title,
            // content: this.announcement.content,
            visible: this.announcement.visible
          }
        }
        if (this.contestID) {
          data.contest_id = this.contestID
          funcName = this.mode === 'edit' ? 'updateContestAnnouncement' : 'createContestAnnouncement'
        } else {
          funcName = this.mode === 'edit' ? 'updateClassify' : 'createClassify'
        }
        api[funcName](data).then(res => {
          this.showEditAnnouncementDialog = false
          this.init()
        }).catch()
      },
      // 子类提交
      submitSubclass (data = undefined) {
        let funcName = ''
        if (!data.title) {
          data = {
            id: parseInt(this.subclassId), // 父id
            parent: this.currentAnnouncementId,  // 子id
            name: this.announcement.title,
            // content: this.announcement.content,
            visible: this.announcement.visible
          }
        }
        if (this.contestID) {
          data.contest_id = this.contestID
          funcName = this.mode === 'edit' ? 'updateContestAnnouncement' : 'createContestAnnouncement'
        } else {
          funcName = this.mode === 'edit' ? 'updateSubclass' : 'createSubclass'
        }
        console.log(data.parent, data.id)
        console.log(data.name)
        api[funcName](data).then(res => {
          this.showEditSubclassDialog = false
          this.init()
        }).catch()
      },
      // 删除分类
      deleteClassify (announcementId) {
        this.$confirm('Are you sure you want to delete this Classify?', 'Warning', {
          confirmButtonText: 'Delete',
          cancelButtonText: 'Cancel',
          type: 'warning'
        }).then(() => {
          // then 为确定
          this.loading = true
          let funcName = this.contestID ? 'deleteContestAnnouncement' : 'deleteClassify'
          api[funcName](announcementId).then(res => {
            this.loading = true
            this.init()
          })
        }).catch(() => {
          // catch 为取消
          this.loading = false
        })
      },
      // 删除子类
      deleteSubclass (announcementId) {
        this.$confirm('Are you sure you want to delete this Subclass?', 'Warning', {
          confirmButtonText: 'Delete',
          cancelButtonText: 'Cancel',
          type: 'warning'
        }).then(() => {
          // then 为确定
          console.log(announcementId)
          this.loading = true
          let funcName = this.contestID ? 'deleteContestAnnouncement' : 'deleteSubclass'
          api[funcName](announcementId).then(res => {
            this.loading = true
            this.init()
          })
        }).catch(() => {
          // catch 为取消
          this.loading = false
        })
      },
      openClassifyDialog (id) {
        this.showEditAnnouncementDialog = true
        if (id !== null) {
          this.currentAnnouncementId = id
          this.announcementDialogTitle = '修改 分类大类'
          this.announcementList.find(item => {
            if (item.id === this.currentAnnouncementId) {
              this.announcement.title = item.name
              this.announcement.visible = true
              this.announcement.content = ''
              this.mode = 'edit'
            }
          })
        } else {
          this.announcementDialogTitle = '新建 分类大类'
          this.announcement.title = ''
          this.announcement.visible = true
          this.announcement.content = ''
          this.mode = 'create'
        }
      },
      openSubclassDialog (id, parentid) {
        this.showEditSubclassDialog = true
        if (parentid == null) {
          this.currentAnnouncementId = id
          this.announcementDialogTitle = '新建 子类'
          this.announcement.title = ''
          this.announcement.visible = true
          this.announcement.content = ''
          this.mode = 'create'
        } else {
          this.currentAnnouncementId = id
          this.announcementDialogTitle = '修改 子类'
          this.subclassId = parentid
          this.subclassList.find(item => {
            if (item.id === this.currentAnnouncementId) {
              this.announcement.title = item.name
              this.announcement.visible = true
              this.announcement.content = ''
              this.mode = 'edit'
            }
          })
        }
      }
    },
    watch: {
      $route () {
        this.init()
      }
    }
  }
</script>

<style lang="less" scoped>
  .title-input {
    margin-bottom: 20px;
  }

  .visible-box {
    margin-top: 10px;
    width: 205px;
    float: left;
  }
</style>
