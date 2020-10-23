<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="addRolesDialogVisible = true">添加角色</el-button>
        </el-col>
      </el-row>
      <el-table :data="roleList" border stripe style="width: 100%">
        <el-table-column type="expand">
          <template v-slot="scope">
            <el-row v-for="(item1, index1) in scope.row.children" :key="item1.id" :class="['bdbottom', !index1 ? 'bdtop' : '', 'vcenter']">
              <el-col :span="5">
                <el-tag closable @close="removeRightById(scope.row, item1.id)">{{ item1.authName }}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <el-col :span="19">
                <el-row v-for="(item2, index2) in item1.children" :key="item2.id" :class="['bdbottom', !index2 ? '' : 'bdtop', 'vcenter']">
                  <el-col :span="6">
                    <el-tag type="success" closable @close="removeRightById(scope.row, item2.id)">{{ item2.authName }}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag type="warning" v-for="(item3, index3) in item2.children" :key="item3.id" closable @close="removeRightById(scope.row, item3.id)">
                      {{ item3.authName }}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index" label="#"> </el-table-column>
        <el-table-column prop="roleName" label="角色名称" width="180">
        </el-table-column>
        <el-table-column prop="roleDesc" label="角色描述" width="180">
        </el-table-column>
        <el-table-column prop="level" label="操作">
          <template v-slot="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)">编辑</el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeRoleById(scope.row.id)">删除</el-button>
            <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightDialog(scope.row)" @close="setRightDialogClosed">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <el-dialog title="添加角色" :visible.sync="addRolesDialogVisible" width="50%">
      <el-form :model="addRolesForm" :rules="addRolesRules" ref="addRolesRef" label-width="100px" @close="addRolesDialogClosed">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addRolesForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addRolesForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addRolesDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRole">确 定</el-button>
      </span>
    </el-dialog>
    <el-dialog title="修改角色" :visible.sync="editRolesDialogVisible" width="50%" @close="editDialogClosed">
      <el-form :model="editRolesForm" :rules="editRolesRules" ref="editRolesRef" label-width="100px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editRolesForm.roleName"></el-input>
        </el-form-item>

        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editRolesForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editRolesDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRole">确 定</el-button>
      </span>
    </el-dialog>
    <el-dialog title="分配权限" :visible.sync="SetRightDialogVisible" width="50%">
      <el-tree :data="rightsList" :props="treeProps" show-checkbox node-key="id" default-expand-all :default-checked-keys="defKeys" ref="treeRef"></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="SetRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      roleList: [],
      rightsList: [],
      defKeys: [],
      roleId: "",
      addRolesDialogVisible: false,
      editRolesDialogVisible: false,
      SetRightDialogVisible: false,
      treeProps: {
        label: "authName",
        children: "children",
      },
      addRolesForm: {},
      editRolesForm: {},
      addRolesRules: {
        roleName: [
          { required: true, message: "请输入角色名称", trigger: "blur" },
          { min: 2, max: 10, message: "请输入3-10个字符", trigger: "blur" },
        ],
        roleDesc: [
          { required: true, message: "请输入角色描述", trigger: "blur" },
          { min: 2, max: 10, message: "请输入2-10个字符", trigger: "blur" },
        ],
      },
      editRolesRules: {
        roleName: [
          { required: true, message: "请输入角色名称", trigger: "blur" },
          { min: 2, max: 10, message: "请输入3-10个字符", trigger: "blur" },
        ],
        roleDesc: [
          { required: true, message: "请输入角色描述", trigger: "blur" },
          { min: 2, max: 10, message: "请输入2-10个字符", trigger: "blur" },
        ],
      },
    };
  },
  created() {
    this.getRoleList();
  },
  methods: {
    async getRoleList() {
      const { data: res } = await this.$axios.get("roles");
      if (res.meta.status != 200)
        return this.$message.error("获取角色列表失败");
      this.roleList = res.data;
    },
    addRole() {
      this.$refs.addRolesRef.validate(async (valid) => {
        if (!valid) return;
        const { data: res } = await this.$axios.post(
          "roles",
          this.addRolesForm
        );
        if (res.meta.status != 201) return this.$message.error("添加角色失败");
        this.$message.success("添加角色成功");
        this.getRoleList();
        this.addRolesDialogVisible = false;
      });
    },
    async editRole() {
      this.$refs.editRolesRef.validate(async (valid) => {
        const { data: res } = await this.$axios.put(
          `roles/${this.editRolesForm.roleId}`,
          this.editRolesForm
        );
        if (res.meta.status != 200) return this.$message.error("修改角色失败");
        this.$message.success("修改角色成功");
        this.getRoleList();
        this.editRolesDialogVisible = false;
      });
    },
    addRolesDialogClosed() {
      this.$refs.addRolesRef.resetFields();
    },
    async showEditDialog(id) {
      this.editRolesDialogVisible = true;
      const { data: res } = await this.$axios.get(`roles/${id}`);
      if (res.meta.status != 200) return this.$message.error("查询用户失败");
      this.editRolesForm = res.data;
    },
    editDialogClosed() {
      this.$refs.editRolesRef.resetFields();
    },
    removeRoleById(id) {
      this.$confirm("此操作将永久删除该文件, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(async () => {
          const { data: res } = await this.$axios.delete(`roles/${id}`);
          if (res.meta.status != 200)
            return this.$message.error("删除角色失败");
          this.$message({
            type: "success",
            message: "删除成功!",
          });
          this.getRoleList();
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },
    removeRightById(role, rightId) {
      this.$confirm("此操作将永久删除该权限, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(async () => {
          const { data: res } = await this.$axios.delete(
            `roles/${role.id}/rights/${rightId}`
          );

          if (res.meta.status != 200)
            return this.$message.error("取消权限失败");
          role.children = res.data;

          this.$message({
            type: "success",
            message: "删除成功!",
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },
    async showSetRightDialog(role) {
      this.roleId = role.id;
      const { data: res } = await this.$axios.get("rights/tree");
      if (res.meta.status != 200)
        return this.$message.error("获取权限数据失败");
      this.rightsList = res.data;
      this.getLeafKeys(role, this.defKeys);
      this.SetRightDialogVisible = true;
    },
    getLeafKeys(node, arr) {
      if (!node.children) return arr.push(node.id);
      node.children.forEach((item) => {
        this.getLeafKeys(item, arr);
      });
    },
    setRightDialogClosed() {
      this.defKeys = [];
    },
    async allotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys(),
      ];
      const idStr = keys.join(",");
      const { data: res } = await this.$axios.post(
        `roles/${this.roleId}/rights`,
        {
          rids: idStr,
        }
      );
      if (res.meta.status != 200) return this.$message.error("分配权限失败");
      this.$message.success("分配权限成功");
      this.getRoleList();
      this.SetRightDialogVisible = false;
    },
  },
};
</script>

<style scoped>
.el-tag {
  margin: 7px;
}

.bdbottom {
  border-bottom: 1px solid #eee;
}

.bdtop {
  border-top: 1px solid #eee;
}

.vcenter {
  display: flex;
  align-items: center;
}
</style>