<template>
  <div class="terminology">
    <header>
      <el-row>
        <el-button size="mini" @click="onAdd">新增</el-button>
      </el-row>
    </header>
    <div class="content">
      <ul>
        <li style="margin:8px;" v-for="(item,index) in terminologyList" :key="item._id">
          {{item.name}}
          <span>
            <el-button size="mini" @click="onEdit(index)">编辑</el-button>
          </span>
        </li>
      </ul>
    </div>

    <el-dialog
      :title="dialogtitle"
      :visible="dialogVisible"
      width="50%"
      center
      @close="onDialogClose"
    >
      <el-row style="minHeight:380px">
        <div style="marginBottom:5px" :key="index" v-for="(item,index) in curKeywordsList">
          <SelectTree :treeData="keywordsList" @treeSelect="onTreeSelect($event,index)"/>
          <el-button size="mini" style="margin-left:10px" @click="onDelTreeSelect(index)">删除</el-button>
        </div>
      </el-row>
      <span slot="footer" class="dialog-footer">
        <el-button @click="onDialogClose">取 消</el-button>
        <el-button type="primary" @click="onConfirmAdd">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import SelectTree from "@/components/SelectTree.vue";

export default {
  name: "terminology",
  components: {
    SelectTree: SelectTree
  },
  data() {
    return {
      terminologyList: [
        { _id: 0, name: "术语1", keywords: ["1", "2"] },
        { _id: 1, name: "术语1", keywords: ["1"] }
      ],
      keywordsList: [{ id: "1", text: "词条1" }, { id: "2", text: "词条2" }],

      //dialog
      dialogVisible: false,
      dialogMode: "add", // 'edit' or 'add'
      dialogIndex: -1,
      curKeywordsList: [{ elementId: "", id: "" }]
    };
  },
  computed: {
    dialogtitle() {
      return "edit" == this.dialogMode ? "编辑术语" : "新增术语";
    }
  },
  methods: {
    onAdd() {
      this.curKeywordsList = [{ elementId: "", id: "" }];
      this.dialogMode = "add";
      this.dialogVisible = true;
    },
    onEdit(index) {
      this.curKeywordsList = this.terminologyList[index].keywords;
      this.dialogMode = "edit";
      this.dialogIndex = index;
      this.dialogVisible = true;
    },
    onDialogClose() {
      this.dialogVisible = false;
    },
    onConfirmAdd() {
      // 调用java接口新增术语并刷新列表
    },
    onTreeSelect(datax, index) {
      console.log(datax, index);
    },
    onDelTreeSelect(index) {
      this.selectList.splice(index, 1);
      console.log(this.selectList);
    }
  }
};
</script>
