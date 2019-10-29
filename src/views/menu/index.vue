<template class="container">
<v-page>
  <template>
    <el-button @click="dialogFormVisible = true, time = null" type="primary" style="margin-top:5px;margin-left: 20px">创 建</el-button>
    <el-button @click="onSubmit()" type="success">提 交</el-button>
  </template>
  <el-table  :data="tableData" style="width: 100%">
    <el-table-column prop="id" label="日期"align="center" show-overflow-tooltip :fit='true' :span="24" >
      <template slot-scope="scope">
        <div @click="modifTime(scope.row.id)">
          {{testForm}}
          <span style="font-size: 12pt;">{{testForm[scope.row.id].key}}</span>
          <span>(星期{{conversion(testForm[scope.row.id].key)}})</span>
        </div>
        <div style="display: inline-block">
          <el-card class="box-card">
            <div slot="header" class="clearfix">
              <span>早上</span>
              <el-button style="float: right; padding: 3px 0" type="text"  @click="addMorning(scope.row.id)">新增</el-button>
            </div>
            <el-form :model="testForm[scope.row.id]" ref="menuForm" label-width="60px" class="demo-dynamic">
              <el-form-item
                v-for="(domain, index) in testForm[scope.row.id].morning"
                :label="'菜名' + (index+1)"
                :key =index
              >
                <el-input v-model="domain.dishes" style="width: 75%;"></el-input><el-button @click.prevent="removeMorning(scope.row.id,domain)">删除</el-button>
              </el-form-item>
            </el-form>
          </el-card>
        </div>
        <div style="display: inline-block">
        <el-card class="box-card">
          <div slot="header" class="clearfix">
            <span>中午</span>
            <el-button style="float: right; padding: 3px 0" type="text" @click="addNooning(scope.row.id)">新增</el-button>
          </div>
          <el-form :model="testForm[scope.row.id]" ref="menuForm" label-width="60px" class="demo-dynamic">
            <el-form-item
              v-for="(domain, index) in testForm[scope.row.id].nooning"
              :label="'菜名' + (index+1)"
              :key =index
            >
              <el-input v-model="domain.dishes" style="width: 75%;"></el-input><el-button @click.prevent="removeNooning(scope.row.id,domain)">删除</el-button>
            </el-form-item>
          </el-form>
        </el-card>
        </div>
        <div style="display: inline-block">
        <el-card class="box-card">
          <div slot="header" class="clearfix">
            <span>晚上</span>
            <el-button style="float: right; padding: 3px 0" type="text" @click="addNight(scope.row.id)">新增</el-button>
          </div>
          <el-form :model="testForm[scope.row.id]" ref="menuForm" label-width="60px" class="demo-dynamic">
            <el-form-item
              v-for="(domain, index) in testForm[scope.row.id].night"
              :label="'菜名' + (index+1)"
              :key =index
            >
              <el-input v-model="domain.dishes" style="width: 75%;"></el-input><el-button @click.prevent="removeNight(scope.row.id,domain)">删除</el-button>
            </el-form-item>
          </el-form>
        </el-card>
        </div>
      </template>
    </el-table-column>
  </el-table>
  <el-dialog title="时间" :visible.sync="dialogFormVisible" width="25%">
    <el-form v-model="time"  class="timeBlock">
      <span class="demonstration">默认</span>
      <el-date-picker
        v-model="time"
        value-format="yyyy-MM-dd"
        type="date"
        placeholder="选择日期">
      </el-date-picker>
    </el-form>
    <div slot="footer" class="dialog-footer">
      <el-button @click="dialogFormVisible = false">取 消</el-button>
      <el-button type="primary" @click=" modif ? dialogFormVisible = false: addDay()">确 定</el-button>
    </div>
  </el-dialog>
</v-page>
</template>

<script>
    import querystring from 'querystring'
    export default {
        name: "index",
        data() {
            return {
                time:null,
                modif:false,
                dialogFormVisible:false,
                result:0,
                menuForm: {
                    domains: [{
                        value: ''
                    }],
                },
                testForm:[
                    // {
                    //     value:[{
                    //         menu:[{
                    //             dishes:''
                    //         }],
                    //         moment:''
                    //     }],
                    //     key:"",
                    // }
                    ],
                tableData:[],
                week:["一","二","三","四","五","六","日"]
            }
        },
        methods: {
            modifTime(val){
                this.modif = true;
                this.dialogFormVisible = true;
                this.$set(this.testForm[val], `key`, this.time);
                this.time = null;

            },
            addDay(){
                this.dialogFormVisible = false
                if(this.result === 7){
                    this.$message({
                        message: '一周只有七天,不允许再添加',
                        type: 'warning'
                    });
                    return
                }
                this.tableData.push({id:this.result})
                this.testForm.push(
                    {
                        morning:[
                        ],
                        nooning: [
                        ],
                        night:[
                        ],
                        key:this.time
                }
                    )
                this.result=this.result+1;
            },
            conversion(val){
                const time=new Date(val);
                return time.getDay();
            },
            onSubmit() {
                console.log('submit!');
                    this.api({
                        url: "/menu/add",
                        method: "POST",
                        contentType:'application/json;charset=utf-8',
                        dataType: "json",
                        data: this.testForm
                    }).then(() => {
                        this.$message({
                            message: '提交成功',
                            type: 'warning'
                        });
                    })
                },
            submitForm(formName) {
                this.$refs[formName].validate((valid) => {
                    if (valid) {
                        alert('submit!');
                    } else {
                        console.log('error submit!!');
                        return false;
                    }
                });
            },
            resetForm(formName) {
                this.$refs[formName].resetFields();
            },
            removeDomain(index,item) {
                var index = this.menuForm.domains.indexOf(item)
                if (index !== -1) {
                    this.menuForm.domains.splice(index, 1)
                }
            },
            removeMorning(index,item){
                var val = this.testForm[index].morning.indexOf(item);
                if (val !== -1) {
                    this.testForm[index].morning.splice(val, 1)
                }
            },
            removeNooning(index,item){
                var val = this.testForm[index].nooning.indexOf(item);
                if (val !== -1) {
                    this.testForm[index].nooning.splice(val, 1)
                }
            },
            removeNight(index,item){
                var val = this.testForm[index].night.indexOf(item);
                if (val !== -1) {
                    this.testForm[index].night.splice(val, 1)
                }

            },
            addDomain() {
                this.menuForm.domains.push({
                    value: '',
                    key: Date.now()
                });
            },
            addMorning(index){
                this.testForm[index].morning.push({
                    dishes:'',
                })
            },
            addNooning(index){
                this.testForm[index].nooning.push({
                    dishes:'',
                })
            },
            addNight(index){
                this.testForm[index].night.push({
                    dishes:'',
                })
            }
        }
    }
</script>

<style >
  .container {
    font-size: 8px;
    padding: 0;
    margin: 0;
  }
  .clearfix:before,
  .clearfix:after {
    display: table;
    content: "";
  }
  .clearfix:after {
    clear: both
  }
  .box-card {
    width: 440px;
  }
  .timeBlock {

  }
</style>
