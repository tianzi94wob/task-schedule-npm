<template>
  <div id="app">
    <el-row type="flex">
      <el-col :span="18" :offset="3">
        <template>
          <el-tabs value="first" type="card" >
            <el-tab-pane name="first">
              <span slot="label"><i class="el-icon-upload"></i>  集群节点</span>
            </el-tab-pane>
          </el-tabs>
        </template>
      </el-col>
    </el-row>
    <el-row type="flex">
      <el-col :span="18" :offset="3">
        <el-table
                :data="serverTableData"
                style="width: 100%;">
          <el-table-column
                  type="index"
                  label="序号"
                  width="90"
                  align="left">
          </el-table-column>
          <el-table-column
                  prop="serverName"
                  label="节点名称"
                  align="left">
          </el-table-column>
          <el-table-column
                  prop="leader"
                  label="是否调度节点"
                  width="120"
                  align="left">
            <template scope="scope">
              {{ scope.row.leader=='0'?'否':'是'}}
            </template>
          </el-table-column>
        </el-table>
      </el-col>
    </el-row>
    <el-row type="flex">
      <el-col :span="18"><div style="height: 20px;"></div></el-col>
    </el-row>
    <el-row type="flex">
      <el-col :span="18" :offset="3">
        <template>
          <el-tabs value="first" type="card" >
            <el-tab-pane name="first">
              <span slot="label"><i class="el-icon-document"></i>  任务列表</span>
            </el-tab-pane>
          </el-tabs>
        </template>
      </el-col>
    </el-row>
    <el-row :gutter="20">
      <el-col :span="18" :offset="3">
        <el-table
                :data="taskTableData"
                style="width: 100%"
                :default-sort = "{prop: 'lastRunningTime', order: 'descending'}">
          <el-table-column type="expand">
            <template slot-scope="props">
              <el-form label-position="left" inline class="demo-table-expand">
                <el-form-item label="cron表达式">
                  <span>{{ props.row.cronExpression }}</span>
                </el-form-item>
                <el-form-item label="执行次数">
                  <span>{{ props.row.runTimes }}</span>
                </el-form-item>
                <el-form-item label="最近耗时ms">
                  <span>{{ props.row.msc==0?'--':props.row.msc}}</span>
                </el-form-item>
                <el-form-item label="参数">
                  <span>{{ props.row.params?props.row.params:'--'}}</span>
                </el-form-item>
                <el-form-item label="目标bean">
                  <span>{{ props.row.targetBean }}</span>
                </el-form-item>
                <el-form-item label="目标方法">
                  <span>{{ props.row.targetMethod }}</span>
                </el-form-item>
                <el-form-item label="上下文">
                  <span>{{ props.row.prjName }}</span>
                </el-form-item>
                <el-form-item label="执行节点">
                  <span>{{ props.row.currentServer }}</span>
                </el-form-item>
              </el-form>
            </template>
          </el-table-column>

          <el-table-column
                  type="index"
                  label="序号"
                  width="90"
                  align="left">
          </el-table-column>
          <el-table-column
                  prop="jobName"
                  label="任务名"
                  align="left">
          </el-table-column>
          <el-table-column
                  prop="lastRunningTime"
                  :formatter="dateFormat"
                  label="最近执行时间"
                  sortable
                  align="left">
          </el-table-column>
          <el-table-column
                  prop="nextRuningTime"
                  :formatter="dateFormat"
                  label="下次执行时间"
                  sortable
                  align="left">
          </el-table-column>
          <el-table-column
                  prop="type"
                  label="类型"
                  align="left"
                  :filters="[{ text: '持久化任务', value: '2' },{ text: '动态任务', value: '1' } ]"
                  :filter-method="filterTag"
                  filter-placement="bottom-end">
            <template slot-scope="scope">
              <el-tag
                      :type="scope.row.type != '1' ? 'success' : 'primary'"
                      disable-transitions>{{ scope.row.type!='1'?'持久化任务':'动态任务'}}</el-tag>
            </template>
          </el-table-column>
          <el-table-column
                  prop="jobStatus"
                  label="状态"
                  align="left">
            <template scope="scope">
              {{ scope.row.jobStatus=='0'?'未调度':'调度中'}}
            </template>
          </el-table-column>
        </el-table>
      </el-col>
    </el-row>
  </div>
</template>

<style>
  .demo-table-expand {
    font-size: 0;
  }
  .demo-table-expand label {
    width: 90px;
    color: #99a9bf;
  }
  .demo-table-expand .el-form-item {
    margin-right: 0;
    margin-bottom: 0;
    width: 50%;
  }
</style>
<script>
    export default {
        data() {
            return {
                limit:15,
                page: 1,
                formInline: {
                    user: '',
                    region: ''
                },
                serverTableData: [],
                taskTotal: 0,
                taskTableData: []
            }
        },
        methods: {
            filterTag(value, row) {
                return row.type === value;
            },
            dateFormat:function(row, column) {
                var val = row[column.property];
                if (val == undefined) {
                    return "";
                }
                if(val == 0){
                    return "--";
                }
                return this.dateFtt("yyyy-MM-dd hh:mm:ss",new Date(val));
            },
            dateFtt(fmt,date) {
                var o = {
                    "M+" : date.getMonth()+1,                 //月份
                    "d+" : date.getDate(),                    //日
                    "h+" : date.getHours(),                   //小时
                    "m+" : date.getMinutes(),                 //分
                    "s+" : date.getSeconds(),                 //秒
                    "q+" : Math.floor((date.getMonth()+3)/3), //季度
                    "S"  : date.getMilliseconds()             //毫秒
                };
                if(/(y+)/.test(fmt))
                    fmt=fmt.replace(RegExp.$1, (date.getFullYear()+"").substr(4 - RegExp.$1.length));
                for(var k in o)
                    if(new RegExp("("+ k +")").test(fmt))
                        fmt = fmt.replace(RegExp.$1, (RegExp.$1.length==1) ? (o[k]) : (("00"+ o[k]).substr((""+ o[k]).length)));
                return fmt;
            },
            getServerList(current,limit) {
                this.$http.get(
                    '/api/taskSchedule/serverList',
                    {
                        params:{
                            current:current,
                            limit:limit
                        }
                    },
                    {emulateJSON: false}
                ).then((response) => {
                    this.serverTableData = response.data;
                }).catch((response) => {
                    //console.log(response);
                });
            },
            getTaskList(current,limit) {
                this.$http.get(
                    '/api/taskSchedule/taskList',
                    {
                        params:{
                            current:current,
                            limit:limit
                        }
                    },
                    {emulateJSON: false}
                ).then((response) => {
                    this.taskTableData = response.data;
                }).catch((response) => {
                    //console.log(response);
                });
            }
        },
        created: function() {
            this.getServerList(this.page,this.limit);
            this.getTaskList(this.page,this.limit);
        }
    }
</script>

<style>
#app {
  font-family: Helvetica, sans-serif;
  text-align: center;
}

</style>
