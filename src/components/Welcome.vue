<template>
  <div>
    <md-empty-state
      :md-icon="state_msg.icon"
      :md-label="state_msg.label"
      :md-description="state_msg.description">
      <md-button class="md-primary md-raised" @click="doBtn">{{state_msg.btn_info}}</md-button>
    </md-empty-state>
    <md-dialog-prompt
      @md-confirm="addGroup"
      :md-active.sync="add_group_dialog_status"
      v-model="group_code"
      md-title="输入你的群组邀请码"
      md-input-maxlength="32"
      md-input-placeholder="在此输入..."
      md-confirm-text="完成"
      md-cancel-text="取消"/>
    <create-group :show="show_create_dialog" @cancel="show_create_dialog=false" @finish="finish"/>
  </div>
</template>

<script>
  import {Student} from "@/api";
  import {Post} from '@/http';
  import CreateGroup from "@/components/CreateGroup";

  export default {
    name: 'Welcome',
    components: {CreateGroup},
    data: () => ({
      auto_close_dialog: false,
      add_group_dialog_status: false,
      group_code: '',
      show_create_dialog: false,
      state_msg: {},
    }),
    methods: {
      addGroup() {
        if (this.group_code === '') {
          return;
        }
        let that = this;
        Post(Student().addGroup)
          .withURLSearchParams({'code': this.group_code})
          .withSuccessCode(201)
          .withErrorStartMsg('加入失败：')
          .do(response => {
            that.$toasted.success('加入成功', {
              position: "top-right",
              icon: 'check',
              duration: 5000
            });
            that.$store.commit('have_groups');
            that.group_code = '';
            that.$router.push("un_done");
          });
      },
      doBtn() {
        if (this.$user.user_is_student) {
          this.add_group_dialog_status = true
        } else {
          this.show_create_dialog = true;
        }
      },
      finish() {
        this.$router.push("group");
      }
    },
    created() {
      if (this.$user.user_is_student) {
        this.state_msg = {
          icon: 'cloud_upload',
          label: '提交你的第一份作业',
          description: '点击下方按钮，输入你的群组邀请码并进行群组确认来提交你的第一份作业',
          btn_info: '加入群组'
        };
      } else {
        this.state_msg = {
          icon: 'group_add',
          label: '创建你的第一个群组',
          description: '欢迎您，' + this.$user.loginName + '。点击下方按钮，创建第一个群组。',
          btn_info: '创建群组'
        };
      }
    }
  }
</script>

<style scoped>

</style>
