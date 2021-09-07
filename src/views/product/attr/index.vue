
<template>
  <div>
    <!-- 三级联动 -->
    <el-card shadow="always">
      <CategorySelector @handlerCategory="handlerCategory" :isShowList=isShowList />
    </el-card>
    <el-card shadow="always" style="margin-top: 20px">
      <div v-show="isShowList">
        <el-button
          type="primary"
          size="medium"
          icon="el-icon-plus"
          :disabled="!category3Id"
          @click="showAddDiv"
          >添加属性</el-button
        >
        <el-table border style="width: 100%; margin-top: 20px" :data="attrList">
          <el-table-column type="index" label="序号" align="center" width="80">
          </el-table-column>
          <el-table-column prop="attrName" label="属性名称" width="150">
          </el-table-column>
          <el-table-column label="属性值列表" width="width">
            <template slot-scope="{ row }">
              <el-tag
                type="success"
                v-for="attrValue in row.attrValueList"
                :key="attrValue.id"
              >
                {{ attrValue.valueName }}
              </el-tag>
            </template>
          </el-table-column>
          <el-table-column label="操作" width="width">
            <template slot-scope="{ row }">
              <HintButton
                type="warning"
                icon="el-icon-edit"
                title="修改"
                size="mini"
                @click="showUpdateDiv(row)"
              ></HintButton>
              <el-popconfirm :title="`确定删除${row.attrName}吗？`" @onConfirm="deleteAttr(row)">
                <HintButton
                  slot="reference"
                  type="danger"
                  icon="el-icon-delete"
                  title="删除属性"
                  size="mini"
                ></HintButton>
              </el-popconfirm>
            </template>
          </el-table-column>
        </el-table>
      </div>
      <div v-show="!isShowList">
        <el-form :inline="true" :model="attrForm">
          <el-form-item label="属性名">
            <el-input
              placeholder="请输入属性名"
              v-model="attrForm.attrName"
            ></el-input>
          </el-form-item>
        </el-form>
        <el-button type="primary" icon="el-icon-plus" @click="addAttrValue"
        :disabled="!attrForm.attrName"
          >添加属性值</el-button
        >
        <el-button @click="isShowList = true">取消</el-button>
        <el-table
          border
          style="width: 100%; margin-top: 20px"
          :data="attrForm.attrValueList"
        >
          <el-table-column
            type="index"
            label="序号"
            align="center"
            width="80"
          ></el-table-column>
          <el-table-column label="属性值名称" align="center">
            <template slot-scope="{ row, $index }">
              <el-input
                :ref="$index"
                v-if="row.isEdit"
                v-model="row.valueName"
                placeholder="请输入属性值名称"
                @blur="toLook(row)"
                @keyup.enter.native="toLook(row)"
              ></el-input>
              <span
                style="display: block; width: 100%; height: 100%"
                @click="toEdit(row, $index)"
                v-else
                >{{ row.valueName }}</span
              >
            </template>
          </el-table-column>
          <el-table-column label="操作" align="center">
            <template slot-scope="{ row, $index }">
              <el-popconfirm
                :title="`确定删除${row.valueName}吗？`"
                @onConfirm="attrForm.attrValueList.splice($index, 1)"
              >
                <HintButton
                  slot="reference"
                  type="danger"
                  icon="el-icon-delete"
                  title="删除属性值"
                  size="mini"
                ></HintButton>
              </el-popconfirm>
            </template>
          </el-table-column>
        </el-table>
        <el-button type="primary" @click="save" :disabled="attrForm.attrValueList.length<1">保存</el-button>
        <el-button @click="isShowList = true">取消</el-button>
      </div>
    </el-card>
  </div>
</template>

<script>
import cloneDeep from "lodash/cloneDeep";
export default {
  data() {
    return {
      category1Id: "",
      category2Id: "",
      category3Id: "",
      attrList: [],
      isShowList: true,
      attrForm: {
        attrName: "",
        attrValueList: [],
        categoryId: 0,
        categoryLevel: 3,
      },
    };
  },
  name: "attr",
  methods: {
    handlerCategory({ categoryId, level }) {
      if (level === 1) {
        this.category1Id = categoryId;
        //子组件重新选择1级，清空父组件当中的23级和属性列表
        this.category2Id = "";
        this.category3Id = "";
        this.attrList = [];
      } else if (level === 2) {
        this.category2Id = categoryId;
        //子组件重新选择2级，清空父组件当中的3级和属性列表
        this.category3Id = "";
        this.attrList = [];
      } else if (level === 3) {
        this.category3Id = categoryId;
        //发请求拿平台属性的列表数据
        this.getAttrList();
      }
    },
    async getAttrList() {
      let { category1Id, category2Id, category3Id } = this;
      const result = await this.$API.attr.getList(
        category1Id,
        category2Id,
        category3Id
      );
      if (result.code === 200) {
        this.attrList = result.data;
      }
    },
    showAddDiv() {
      this.isShowList = false;
      this.attrForm = {
        attrName: "",
        attrValueList: [],
        categoryId: this.category3Id,
        categoryLevel: 3,
      };
    },
    //添加属性
    addAttrValue() {
      this.attrForm.attrValueList.push({
        attrId: this.attrForm.id,
        valueName: "",
        isEdit: true, //标志这个属性值模式是编辑模式
      });
      this.$nextTick(() => {
        this.$refs[this.attrForm.attrValueList.length - 1].focus();
      });
    },
    showUpdateDiv(row) {
      this.isShowList = false;
      this.attrForm = cloneDeep(row);
      //确保添加的属性是响应式
      this.attrForm.attrValueList.forEach((item) =>
        this.$set(item, "isEdit", false)
      );
    },
    //变为查看模式
    toLook(row) {
      if (row.valueName.trim() === "") {
        row.valueName = "";
        return;
      }
      let isRepect = this.attrForm.attrValueList.some((item) => {
        if (item !== row) return item.valueName === row.valueName;
      });
      if (isRepect) {
        this.$message.info("输入的属性值不能重复");
        row.valueName = "";
        return;
      }
      row.isEdit = false;
    },
    //变为编辑模式
    toEdit(row, index) {
      row.isEdit = true;
      this.$nextTick(() => {
        this.$refs[index].focus();
      });
    },
    //保存
    async save() {
      let attr = this.attrForm;
      attr.attrValueList = attr.attrValueList.filter((item) => {
        if (item.valueName !== "") {
          delete item.isEdit;
          return true;
        }
      });
      if (attr.length === 0) {
        this.$message({
          message: "属性值列表必须有属性值才能保存",
          type: "info",
        });
        return;
      }
      try {
        await this.$API.attr.addOrUpdate(attr);
        this.$message.success("保存成功");
        this.isShowList = true;
        this.getAttrList();
      } catch (error) {
        this.$message({
          message: "保存属性失败",
          type: "error",
        });
      }
    },
    //删除属性
    async deleteAttr(row){
      try {
        await this.$API.attr.deleteAttr(row.id)
        this.$message.success('删除成功')
        this.getAttrList()
      } catch (error) {
        this.$message({
          message: '删除失败',
          type: 'error'
        });
      }
    }
  },
};
</script>


