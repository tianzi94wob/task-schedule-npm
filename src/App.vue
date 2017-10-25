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
                  width="90">
          </el-table-column>
          <el-table-column
                  prop="serverName"
                  label="节点名称">
          </el-table-column>
          <el-table-column
                  prop="leader"
                  label="是否调度节点"
                  width="120">
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
                style="width: 100%">
          <el-table-column
                  type="index"
                  label="序号"
                  width="90">
          </el-table-column>
          <el-table-column
                  prop="type"
                  label="类型">
            <template scope="scope">
              {{ scope.row.type!='1'?'持久化任务':'普通任务'}}
            </template>
          </el-table-column>
          <el-table-column
                  prop="targetBean"
                  label="目标bean"
                  width="120">
          </el-table-column>
          <el-table-column
                  prop="targetMethod"
                  label="目标方法"
                  width="120">
          </el-table-column>
          <el-table-column
                  prop="params"
                  label="参数">
            <template scope="scope">
              {{ scope.row.params?scope.row.params:'--'}}
            </template>
          </el-table-column>
          <el-table-column
                  prop="cronExpression"
                  label="cron表达式">
          </el-table-column>
          <el-table-column
                  prop="currentServer"
                  label="执行节点">
          </el-table-column>
          <el-table-column
                  prop="runTimes"
                  label="执行次数">
          </el-table-column>
          <el-table-column
                  prop="lastRunningTime"
                  :formatter="dateFormat"
                  label="最近执行时间">
          </el-table-column>
          <el-table-column
                  prop="msc"
                  label="最近耗时ms">
            <template scope="scope">
              {{ scope.row.msc==0?'--':scope.row.msc}}
            </template>
          </el-table-column>
          <el-table-column
                  prop="nextRuningTime"
                  :formatter="dateFormat"
                  label="下次执行时间">
          </el-table-column>
          <el-table-column
                  prop="jobStatus"
                  label="状态">
            <template scope="scope">
              {{ scope.row.jobStatus=='0'?'未调度':'调度中'}}
            </template>
          </el-table-column>
        </el-table>
      </el-col>
    </el-row>
  </div>
</template>

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
