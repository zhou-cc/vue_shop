<template>
    <div>
        <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

        <!-- 卡片视图区域 -->
    <el-card>
        <!--    添加角色按钮区域 -->
        <el-row>
            <el-col>
                <el-button type="primary" @click="addRoleVisible = true">添加角色</el-button>
            </el-col>
        </el-row>

        <!-- 角色列表区域 -->
        <el-table :data="roleList" border stripe>
            <!-- 展开列 -->
            <el-table-column type="expand">
                <template slot-scope="scope">
                    <el-row :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']" v-for="(item1,i1) in scope.row.children" :key="item1.id">
                        <!-- 渲染一级权限 -->
                        <el-col :span="5">
                        <el-tag closable @close="removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
                        <i class="el-icon-caret-right"></i>
                        </el-col>
                        <!-- 渲染二、三级权限 -->
                        <el-col :span="19">
                            <!-- 通过for循环，嵌套渲染二级权限 -->
                            <el-row :class="[i2 === 0 ? '' : 'bdtop', '', , 'vcenter']" v-for="(item2, i2) in item1.children" :key="item2.id">
                                <el-col :span="6">
                                    <el-tag type="success" closable @close="removeRightById(scope.row, item2.id)">{{item2.authName}}</el-tag>
                                    <i class="el-icon-caret-right"></i>
                                </el-col>
                                <!-- 通过for循环，嵌套渲染三级权限 -->
                                <el-col :span="18">
                                    <el-tag type="warning" v-for="(item3, i3) in item2.children" :key="item3.id" closable @close="removeRightById(scope.row, item3.id)">
                                        {{item3.authName}}
                                    </el-tag>
                                </el-col>
                            </el-row>
                        </el-col>
                    </el-row>

                    <!-- <pre>
                        {{scope.row}}
                    </pre> -->
                </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column label="角色名称" prop="roleName"></el-table-column>
            <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
            <el-table-column label="操作" width="300px">
                <template slot-scope="scope">
                  <!-- 编辑按钮 -->
                  <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)">编辑</el-button>
                  <!-- 删除按钮 -->
                  <el-button type="danger" icon="el-icon-delete" size="mini" >删除</el-button>
                  <!-- 分配权限按钮 -->
                  <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightDialog(scope.row)">分配权限</el-button>
                </template>
            </el-table-column>
        </el-table>
    </el-card>

    <!-- 添加角色的对话框 -->
    <el-dialog title="添加角色" :visible.sync="addRoleVisible" width="50%" @close="addRoleClosed">
    <!-- 内容主体区 -->
    <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
            <el-form-item label="名称" prop="roleName">
            <el-input v-model="addForm.roleName"></el-input>
            </el-form-item>
            <el-form-item label="描述" prop="roleDesc">
            <el-input v-model="addForm.roleDesc"></el-input>
            </el-form-item>
    </el-form>
    <!-- 底部按钮区 -->
    <span slot="footer" class="dialog-footer">
        <el-button @click="addRoleVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRole">确 定</el-button>
    </span>
    </el-dialog>

    <!-- 编辑角色的对话框 -->
    <el-dialog title="编辑角色" :visible.sync="editRoleDialogVisible" width="50%" @close="addRoleClosed">
        <el-form :model="addForm" :rules="editRoleFormRules" ref="addFormRef" label-width="70px">
            <el-form-item label="名称" prop="roleName">
            <el-input v-model="addForm.roleName"></el-input>
            </el-form-item>
            <el-form-item label="描述" prop="roleDesc">
            <el-input v-model="addForm.roleDesc"></el-input>
            </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
        <el-button @click="editRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRoleInfo">确 定</el-button>
        </span>
    </el-dialog>

    <!-- 分配权限的对话框 -->
    <el-dialog title="分配权限" :visible.sync="setRightdialogVisible" width="50%" @close="setRightDialogClosed">
        <!-- 树形控件 -->
         <el-tree :data="rightslist" :props="treeProps" show-checkbox default-expand-all node-key="id" :default-checked-keys="defKeys" ref="treeRef"></el-tree>
         <span slot="footer" class="dialog-footer">
         <el-button @click="setRightdialogVisible = false">取 消</el-button>
         <el-button type="primary" @click="allotRights">确 定</el-button>
         </span>
    </el-dialog>

    </div>
</template> 

<script>
export default {
    data(){
        return{
            // 控制分配权限对话框的显示与隐藏
            setRightdialogVisible:false,
            // 所有角色列表数据
            roleList:[],
            // 树形控件的属性绑定对象
            treeProps:{
                label:'authName',
                children:'children'
            },
            // 默认选中的节点id值数组
            defKeys:[ ],
            // 当前即将分配权限的角色id
            roleId:'',
            // 控制添加角色对话框的显示与隐藏
            addRoleVisible:false,
            // 所有权限的数据
            rightslist:[],
            // 添加角色的表单数据
            addForm:{ 
                roleName:'',
                roleDesc:''
            },
            // 添加角色的验证规则对象
            addFormRules:{
                roleName:[
                {required:true,
                message:'请输入角色名称', trigger:'blur'},
                {min:2,
                max:10,
                message:'角色名称的长度在2～10个字符之间',
                trigger:'blur'}
            ],
                roleDesc:[
                {required:true,
                message:'请输入角色描述', trigger:'blur'},
                {min:2,
                max:10,
                message:'角色描述的长度在2～10个字符之间',
                trigger:'blur'}
            ]
            },
            // 控制编辑角色对话框的显示与隐藏
            editRoleDialogVisible:false,
            // 修改表单的验证规则对象
            editRoleFormRules:{
                roleName: [
                    { required: true, message: '请输入角色名称', trigger: 'blur' }],
                roleDesc: [
                    { required: true, message: '请输入角色描述', trigger: 'blur' }]
           },
        }
    },
    created(){
        // 获取所有的角色
        this.getRolesList();
    },
    methods:{
        // 获取所有角色列表
        async getRolesList(){
            const { data:res } = await this.$http.get('roles');
            // console.log(res);
            if(res.meta.status !== 200){
                return this.$message.error('获取角色列表失败！');
            }
            // 如果失败，将获取到的列表数据保存到rolesList
            this.roleList = res.data;
            // console.log(this.roleList);
        },    
        // 监听修改角色对话框的关闭事件
        addRoleClosed() {
            this.$refs.addFormRef.resetFields();
        },
        addRole() {
            this.$refs.addFormRef.validate(async valid => {
                // console.log(valid);
                if(!valid) return;
                // 发起添加角色的网络请求
                const {data:res} = await this.$http.post('roles', this.addForm);
                if(res.meta.status !== 201) {
                    this.$message.error('添加角色失败！');
                }
                this.$message.success('添加角色成功～');
                //  隐藏添加角色的对话框
                this.addRoleVisible = false;
                // 重新获取用户
                this.getRolesList();
            })
        },
        // 展示编辑角色的对话框
        async showEditDialog(id){
            console.log(id);
            const {data:res} = await this.$http.get('roles/' + id);
            if(res.meta.status !== 200) {
                return this.$message.error('查询角色id失败！');
            }
            this.addForm = res.data;
            this.editRoleDialogVisible = true;
        },
        //    监听编辑角色对话框的关闭事件
        editRoleDialogClosed(){
            this.$refs.addFormRef.resetFields();
        },
        // 编辑用户信息并提交
        editRoleInfo(){
            this.$refs.addFormRef.validate(async valid =>{
                // console.log(valid);
                if(!valid) return;
                const{data:res} = await this.$http.put('roles/' + this.addForm.roleId,
                {roleName:this.addForm.roleName,
                roleDesc:this.addForm.roleDesc});
                if(res.meta.status !== 200){
                    return this.$message.error('更新用户失败！');
                }
                // 关闭对话框
                this.editRoleDialogVisible = false;
                // 刷新数据列表
                this.getRolesList();
                // 提示修改成功
                this.$message.success('编辑用户成功～～');
            })
        },
        // 根据ID删除对应的权限
        async removeRightById(role,rightId){
            // 弹框提示用户是否删除
            const confirmResult= await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
            }).catch(err => err)
           if(confirmResult !== 'confirm') {
               return this.$message.info('取消了删除！');
           }
        //    console.log('确认了删除！');
            const {data:res} = await this.$http.delete(`roles/${role.id}/rights/${rightId}`);
            if(res.meta.status !== 200) {
                return this.$message.error('删除权限失败！');
            }
                // this.getRolesList();
                role.children = res.data
        },
        // 展示分配权限对话框
        async showSetRightDialog(role){
            this.roleId = role.id;
            // 获取所有权限的数据
            const {data:res} = await this.$http.get('rights/tree');
            if(res.meta.status !== 200) {
                return this.$message.error('获取权限数据失败！');
            }
            // 把获取到的权限数据保存到data中
            this.rightslist = res.data;
            console.log(this.rightslist);

            // 递归获取三级节点的id
            this.getLeafKeys(role, this.defKeys);

            this.setRightdialogVisible = true;
        },
        // 通过递归的形式，获取角色下所有三级权限的id，并保存到 defkeys 数组中
        getLeafKeys(node, arr) {
            // node 是用来判断是否是三级节点，即是否包含children属性，arr 数组用来保存
            if(!node.children) {
                // 是三级节点，就将 id push到数组里
                return arr.push(node.id);
            }
            // 递归开始
            node.children.forEach(item =>
            this.getLeafKeys(item, arr));
        },
        // 监听分配权限对话框的关闭事件
        setRightDialogClosed(){
            this.defKeys = [];
        },
        // 点击为角色分配权限
        async allotRights(){
            const keys = [
                // 复习es6语法：展开运算符，把数组转化为逗号分隔的字符串
                ...this.$refs.treeRef.getCheckedKeys(),
                ...this.$refs.treeRef.getHalfCheckedKeys()
            ]
            // console.log(keys);
            const idStr = keys.join(',');
            const {data:res} = await this.$http.post(`roles/${this.roleId}/rights`, {rids: idStr});
            if(res.meta.status !== 200) {
                return this.$message.error('更新权限失败!');
            }
            this.$message.success('更新权限成功！');
            this.getRolesList();
            this.setRightdialogVisible = false;
        }
    }
}
</script>

<style lang="less" scoped>
.el-tag{
    margin: 7px ;
}
.bdtop{
    border-top: 1px solid #eee;
}
.bdbottom{
    border-bottom: 1px solid #eee;
}
.vcenter{
    display: flex;
    align-items: center;
}
</style>