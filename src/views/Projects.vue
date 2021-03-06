<template>
  <div class="projects">
    <!-- <h3>Current: {{ projects.project.name}}</h3> -->
    <el-table :data="allProjects" size="small" v-loading="loading" @cell-mouse-enter="onHover">
      <el-table-column label="Name" prop="name">
        <template slot-scope="scope">
          <span @click.stop="onEditName(scope.row)">
            <span v-if="clicked.id === scope.row.id">
              <el-row :gutter="10">
                <el-col :span="18">
                  <el-input size="mini" v-model="clicked.name"></el-input>
                </el-col>
                <el-col :span="6">
                  <div class="name-actions">
                    <span @click.stop="onUpdateName" class="name-actions name-actions--success">
                      <i class="el-icon-check"></i>
                    </span>
                    <span @click.stop="clicked = {}" class="name-actions name-actions--cancel">
                      <i class="el-icon-close"></i>
                    </span>
                  </div>
                </el-col>
              </el-row>
            </span>
            <span v-else>
              <span v-if="projects.project.id === scope.row.id">
                <strong>{{ scope.row.name }}</strong>
              </span>
              <span v-else>{{ scope.row.name }}</span>
            </span>
          </span>
          <!-- <span v-if="projects.project.id === scope.row.id">
            <strong>{{ scope.row.name }}</strong>
          </span>
          <span v-else>{{ scope.row.name }}</span>-->
        </template>
      </el-table-column>
      <el-table-column label="Actions" width="220">
        <template slot-scope="scope">
          <el-button type="text" size="mini" @click="onLoad(scope.row)">
            Load
            <i class="el-icon-upload2"></i>
          </el-button>
          <el-button type="text" size="mini" @click="download(scope.row)">
            Download
            <i class="el-icon-download"></i>
          </el-button>
          <el-button type="text" size="mini" @click="onDelete(scope.row.id)">
            Delete
            <i class="el-icon-delete"></i>
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <div class="project-actions">
      <el-button size="small" @click="onImportProject">Import project</el-button>
      <el-button size="small" type="success" @click="onNew">New project</el-button>
      <input style="display: none;" type="file" @change="onFileSelected" ref="fileInput">
    </div>
    <div class="desc">
      <p>Your changes and your projects only get stored in your browser's cache. Clearing your browsing data will result in losing your projects.</p>
    </div>
  </div>
</template>

<script>
import { mapState } from 'vuex'
import { deleteProjectById } from '../db/indexedDB.js'

export default {
  name: '',

  data () {
    return {
      loading: false,
      dbname: '',
      version: 1,
      db: '',
      store: '',
      index: '',
      tx: '',
      hovered: {},
      clicked: {},
      selectedFile: null
    }
  },

  computed: {
    ...mapState(['basic', 'options', 'template', 'projects']),
    allProjects () {
      return this.projects.projects.filter(item => item.id !== 'currentProject')
    }
  },

  async created () {
    this.$ga.page(this.$router)
    this.$store.dispatch('getProjects')
  },

  methods: {
    async onNew () {
      await this.$store.dispatch('newProject')
      this.gaEventClick('new project')
    },
    async onDelete (id) {
      try {
        await this.$confirm('This will permanently delete the file. Continue?', 'Warning', {
          confirmButtonText: 'OK',
          cancelButtonText: 'Cancel',
          type: 'warning'
        })
        await deleteProjectById(id)
        await this.$store.dispatch('getProjects')
        this.gaEventClick('delete project')
        if (this.projects.projects.length > 0) {
          await this.$store.dispatch('setCurrentProject')
        } else {
          this.$store.dispatch('newProject')
        }
      } catch (err) {

      }
    },

    async onLoad (project) {
      this.$store.dispatch('setProject', project)
      this.gaEventClick('load project')
    },
    onUpdate () {
      this.$store.dispatch('updateProject', this.projects.project)
    },
    onHover (v) {
      this.hovered = { ...v }
    },
    onEditName (v) {
      this.clicked = { ...v }
    },
    onUpdateName () {
      this.$store.dispatch('updateProject', this.clicked)
      this.clicked = {}
    },
    download (project) {
      const data = JSON.stringify(project)
      // return alert(data)
      const a = document.createElement('a')
      const file = new Blob([data], { type: 'application/json' })
      a.href = URL.createObjectURL(file)
      a.download = project.name
      a.click()
      this.gaEventClick('download project')
    },
    onFileSelected (e) {
      const file = e.target.files[0]
      const fl = new FileReader()
      fl.readAsText(file)
      fl.onload = e => {
        this.$store.dispatch('importProject', JSON.parse(e.target.result))
        // Удаление загруженного файла из интупа
        this.$refs.fileInput.value = ''
      }
    },
    onImportProject () {
      this.$refs.fileInput.click()
      this.gaEventClick('import project')
    }

  }
}
</script>

<style lang="scss">
@import '../assets/scss/variables';

.project-actions {
  margin-top: 20px;
  text-align: right;
}
.name-actions {
  padding: 2px 3px;
  cursor: pointer;
  border-radius: 2px;
  + .name-actions {
    margin-left: 2px;
  }
  &--success {
    background-color: $color-success;
    i {
      color: #fff;
    }
  }
  &--cancel {
    background-color: $color-danger;
    i {
      color: #fff;
    }
  }
}
</style>
