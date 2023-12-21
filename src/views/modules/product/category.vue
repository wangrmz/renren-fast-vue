<!--  -->
<template>
  <div>
    <el-tree :data="menus" :props="defaultProps" :expand-on-click-node="false" show-checkbox node-key="catId"
      :default-expanded-keys="expandeKey">
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button v-if="node.level <= 2" type="text" size="mini" @click="() => append(data)">Append</el-button>
          <el-button v-if="node.childNodes.length == 0" type="text" size="mini"
            @click="() => remove(node, data)">Delete</el-button>
        </span>
      </span>
    </el-tree>
    <!-- 对话框 -->
    <el-dialog title="提示" :visible.sync="dialogVisible" width="30%">
      <!-- model是表单的数据对象，element官网属性说明 -->
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCategory">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';

export default {
  //import引入的组件需要注入到对象中才能使用
  components: {},
  data() {
    return {
      category: { name: "", parentCid: 0, catLevel: 0, showStatus: 1, sort: 0 },//三级分类对象
      dialogVisible: false,// 默认关闭
      menus: [],
      expandeKey: [],
      defaultProps: {
        children: "children",
        label: "name",
      },
    };
  },
  methods: {
    handleNodeClick(data) {
      console.log(data);
    },
    /**
     * 获取所有菜单
     */
    getMenus() {
      // 发送请求
      this.$http({
        // product/category/list/tree
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        console.log("成功获取到菜单数据...", data.data);
        // TODO 获取到菜单数据
        this.menus = data.data;
      });
    },

    /**
     * 添加节点
     */
    append(data) {
      console.log("append", data);
      // 打开对话框
      this.dialogVisible = true;
      // 计算初始化caterory的其他值
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
    },
    /**
     * 添加三级分类
     */
    addCategory() {
      console.log("提交的三级分类数据", this.category);
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      }).then(({ data }) => {
        // success
        this.$message({
          message: "菜单保存成功",
          type: "success",
        });
        //关闭对话框
        this.dialogVisible = false;
        // 重新加载新的菜单
        this.getMenus();
        // 设置需要默认展示的菜单
        this.expandeKey = [this.category.parent.data.catId];
      })
        .catch(() => {
          // 取消
        });

    },


    /**
     * 删除节点
     */
    remove(node, data) {
      var ids = [data.catId];
      console.log(data.name);

      // 添加弹框提示
      this.$confirm("是否删除【" + data.name + "】菜单", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          // 确定删除
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false),
          }).then(({ data }) => {
            // success
            // console.log("删除成功");
            this.$message({
              message: "菜单删除成功",
              type: "success",
            });
            // TODO 展开父节点 expandeKey
            // 重新加载新的菜单
            this.getMenus();
            // 设置需要默认展示的菜单
            // 真正的数据在data中存放
            this.expandeKey = [node.parent.data.catId];
          });
        })
        .catch(() => {
          // 取消
        });

      console.log("remove", node, data);
    },
  },
  //监听属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {},

  //生命周期 - 创建完成（可以访问当前this实例）
  created() {
    this.getMenus();
  },
  //生命周期 - 挂载完成（可以访问DOM元素）
  mounted() { },
  beforeCreate() { }, //生命周期 - 创建之前
  beforeMount() { }, //生命周期 - 挂载之前
  beforeUpdate() { }, //生命周期 - 更新之前
  updated() { }, //生命周期 - 更新之后
  beforeDestroy() { }, //生命周期 - 销毁之前
  destroyed() { }, //生命周期 - 销毁完成
  activated() { }, //如果页面有keep-alive缓存功能，这个函数会触发
};
</script>
<style scoped></style>