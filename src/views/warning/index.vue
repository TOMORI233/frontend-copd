<template>
  <div class="warning-container">
    <div class="info">
		<span>共&nbsp;{{this.patientTotal}}&nbsp;</span>位预警患者</span>
		<span>&nbsp;</span>
		<el-button type="primary" :loading="isLoading" @click="load()">{{LoadingText}}</el-button>
	</div>
	<div class="body" v-loading="isSearching">
		<!--<div v-if="warningPatientList.length === 0">
			<span v-if="isError" class="error">
			  <i class="el-icon-error"></i>
			  <div style="color:red">{{errSummary}}</div>
			  <div style="color:red">{{errDetail}}</div>
			</span>
			<span v-else>无内容显示</span>
		</div>-->
		<div>
			<div class="card" v-for="item,index in warningPatientList" :id="item.patientID">
			    <el-card>
				    <div slot="header" class="clearfix">
						<svg-icon v-show="item.sex==1" icon-class="man" style="font-size:30px;color:blue"></svg-icon>
						<svg-icon v-show="item.sex==2" icon-class="woman" style="font-size:30px;color:deeppink"></svg-icon>
						<span class="nameSexAge">{{item.patientName}}</span>
						<span class="nameSexAge">{{item.sex==1?'男':'女'}}</span>
						<span class="nameSexAge">{{dateToAge(item.dateOfBirth)}}岁</span>
						<span></span>
					</div>
					<div class="WarningInfoContainer">
						<div class="left">
							<span class="blueblock"></span>
							预警信息：
						</div>
						<div class="right">
							<div v-if="item.alertItemList.length>0">
								<div class="alertItem" v-for="alertItem in item.alertItemList">
									<span class="el-icon-warning warningIcon"/>
									<span class="alertNameReason">{{alertItem.alertName}}</span>
									<span class="alertNameReason">{{alertItem.alertReason}}</span>
									<span class="alertDateTime">{{alertItem.alertDateTime}}</span>
								</div>
							</div>
						</div>
					</div>
					<div class="MedicationInfoContainer">
						<div class="left">
							<span class="blueblock"></span>
							用药情况：
						</div>
						<div class="right">
							<div v-if="item.drugRecordList!=null">
								<div class="drugItem" v-for="drug in item.drugRecordList">
									<span class="lbl">{{drug.drugName}}</span>
									<span class="content">{{drug.dosage}}</span>
									<span v-if="drug.freq" class="content">{{drug.freq}}</span>
									<span class="content">
										<span v-if="drug.usedDays>0">
											<span>已用</span>
											<span v-if="drug.usedDays<30">{{drug.usedDays}}天</span>
											<span v-else>{{Math.floor(drug.usedDays/30)}}个月</span>
										</span>
										<span v-else>刚开始使用</span>
									</span>
								</div>
							</div>
							<div v-else>
								<span class="content">最近暂无用药记录</span>
							</div>
						</div>
					</div>
					<div class="ManageSituationContainer">
						<div class="left">
							<span class="blueblock"></span>
							管理情况：
						</div>
						<div class="right">
							<div class="row">
								<div v-if="item.manageItem.manageStatus==0">
									<div style="display: flex; align-items: center;">
										<div v-if="item.manageItem.manageClass===0">
											<div>
												<span class="row">高血压管理等级：</span>
												<span v-if="item.manageItem.manageLevel==0">新患者</span>
												<span v-else-if="item.manageItem.manageLevel==1">一级</span>
												<div v-else-if="item.manageItem.manageLevel==2" style="display: inline-block; align-items: center;">
													<span>二级&nbsp;</span>
													<!--<el-tooltip effect="light">
														<div slot="content">
															<span>二级管理第</span>
															<span>{{toWeeks(item.manageItem.manageStartDateTime.replace(/-/g,'/'))}}</span>
															<span>周（第</span>
															
															<span>{{toDays(item.manageItem.manageStartDateTime.replace(/-/g,'/'), new Date())}}</span>
															<span>天）</span>
														</div>
														<img :src="imgMLM4" v-if="toWeeks(item.manageItem.manageStartDateTime.replace(/-/g,'/'))>3" />
														<img :src="imgMLM3" v-else-if="toWeeks(item.manageItem.manageStartDateTime.replace(/-/g,'/'))>2" />
														<img :src="imgMLM2" v-else-if="toWeeks(item.manageItem.manageStartDateTime.replace(/-/g,'/'))>1" />
														<img :src="imgMLW1" v-else/>
													</el-tooltip>-->
												</div>
											</div>
										</div>
									</div>
								</div>
								<div v-else>
									<span>终止管理</span>
								</div>
							</div>
							<div class="row">
								<span class="row">依从度</span>
								<span class="row">
									<star :on="item.manageItem.complianceRate" :off="5-item.manageItem.complianceRate" ></star>
								</span>
							</div>
							<div style="padding-top: 10px;">
								<span>{{"已管理"+item.manageItem.manageDays+"天，共随访"+item.manageItem.followupTimes+"次"}}</span>
								<span v-if="item.manageItem.lastFollowupDate!=null">{{"，上次随访："+toDateText(item.manageItem.lastFollowupDate.replace(/-/g,"/"))+getDaysText(item.manageItem.lastFollowupDays)}}</span>
								<span v-if="item.manageItem.followupDate!=null">{{"，计划随访："+toDateText(item.manageItem.followupDate.replace(/-/g,"/"))+getDaysText(item.manageItem.followupDays)}}</span>
							</div>
							<div class="btn">
								<el-button size="medium" type="primary" @click="openCreateFollowRecord(item.patientID,item.alertItemList)">立即干预</el-button>
								<el-dialog
								  title="新建随访记录"
								  :visible.sync="centerDialogVisible"
								  width="60%"
								  center>
									<el-form ref="form" :model="form" label-width="80px">
										<el-form-item label="随访方式">
											<el-radio v-model="form.followupMethod" label="门诊">门诊</el-radio>
											<el-radio v-model="form.followupMethod" label="家访">家访</el-radio>
											<el-radio v-model="form.followupMethod" label="电话">电话</el-radio>
										</el-form-item>
										<el-form-item label="随访结果">
											<el-radio v-model="form.status" label="0">失访</el-radio>
											<el-radio v-model="form.status" label="1" @change="form.failureReason=''">进行中</el-radio>
											<el-radio v-model="form.status" label="2" @change="form.failureReason=''">有效</el-radio>
										</el-form-item>
										<el-form-item label="随访类型">
											<el-radio v-model="form.followupType" label="常规随访">常规随访</el-radio>
											<el-radio v-model="form.followupType" label="预警干预">预警干预</el-radio>
											<el-radio v-model="form.followupType" :label="form.otherFailureReason">其他</el-radio>
											<el-input
											  v-model="form.otherFailureReason"
											  size="medium"
											  placeholder="请输入其他类型"></el-input>
										</el-form-item>
										<el-form-item label="失访原因">
											<el-input
											  v-model="form.failureReason"
											  placeholder="请输入内容"
											  :disabled="form.status!=0"></el-input>
										</el-form-item>
										<el-form-item label="是否死亡">
											<el-radio v-model="form.death" :label="true">是</el-radio>
											<el-radio v-model="form.death" :label="false" @change="form.deathTime=''">否</el-radio>
										</el-form-item>
										<el-form-item label="死亡时间">
											<el-date-picker
											  :disabled="!form.death"
											  v-model="form.deathTime"
											  type="datetime"
											  placeholder="选择日期时间">
											</el-date-picker>
										</el-form-item>
										<el-form-item label="摘要记录">
											<el-input
											  type="textarea"
											  autosize
											  placeholder="请输入内容"
											  v-model="form.content">
											</el-input>
										</el-form-item>
										
									</el-form>
									<span slot="footer" class="dialog-foowter">
										<el-button @click="centerDialogVisible = false">取 消</el-button>
										<el-button type="primary" @click="submitCreateFollows">确 定</el-button>
									</span>
								</el-dialog>
								<el-popover
									trigger="click"
									width="210"
									placement="bottom-start"
									v-model="item.visible"
									:popper-class="'p'+item.patientID"
									@hide="ignoreReason='重复预警'">
									<p>忽略预警</p>
									<el-select v-model="ignoreSerialNo" style="width: 190px" size="mini" placeholder="请选择">
										<el-option :label="alertItem.alertName" :value="alertItem.serialNo" v-for="alertItem in item.alertItemList" :key="alertItem.serialNo" />
									</el-select>
									<p>忽略预警的原因：</p>
									<el-select v-model="ignoreReason" style="width: 190px" size="mini">
										<el-option label="重复预警" value="重复预警"></el-option>
										<el-option label="最近已随访" value="最近已随访"></el-option>
									</el-select>
									<div style="text-align: center; margin-top: 15px">
										<el-button class="btn-ignore" size="mini" @click="closeCard(item,index)">忽略预警</el-button>
										<el-button class="btn-ignore" size="mini" @click="doClose(item,index)">取消</el-button>
									</div>
									<el-button style="margin-left: 12px" size="medium" slot="reference">忽略预警</el-button>
								</el-popover>
							</div>
						</div>
					</div>
				</el-card>
			</div>
		</div>
	</div>
	<div class="footer">
	  Copyright © 2019 浙江大学
	</div>
  </div>
</template>

<script>
	import Component from 'vue-class-component'
	import BaseComponent from '../../components/BaseComponent'
    import ImgMLW1 from '../../images/data/mlw1.png'
    import ImgMLW2 from '../../images/data/mlw2.png'
    import ImgMLW3 from '../../images/data/mlw3.png'
    import ImgMLW4 from '../../images/data/mlw4.png'
	import Star from '../../components/graphic/Star'
	import {getWarningPatientCount,getWarningPatientList,deleteWarningPatient,createFollowRecord} from '../../api/warningPatientList'
	@Component({
		components: {
			star: Star
		}
	})
	export default class Warning extends BaseComponent {
	  patientTotal = 0
	  patientID = 0
	  isSearching = false
	  isError = false
	  errSummary = ""
	  errDetail = ""
	  isLoading = false
	  LoadingText = "刷新"
	  ignoreReason = "重复预警"
	  warningPatientList=[]
	  centerDialogVisible=false;
	  form={
		followupMethod: '',
		status: '1',
		followupType: '',
		failureReason: '',
		death: false,
		deathTime: '',
		content: '',
		otherFailureReason: '',
	  }
	  selectedPatientID=''
	  selectedAlertItems=[]
	  
	  imgMLM1= ImgMLW1
	  imgMLM2= ImgMLW2
	  imgMLM3= ImgMLW3
	  imgMLM4= ImgMLW4
	  ignoreSerialNo = null;
	  
	  toWeeks(date) {
        let now = new Date();
        let days = this.toDays(date, now);

        let weeks = 1;
        if(days > 21) {
           weeks = 4;
        }
        else if(days > 14) {
           weeks = 3;
        }
        else if(days > 7) {
           weeks = 2;
        }
        return weeks;
      }
	  doClose(item, index) {
		this.ignoreSerialNo=null
		item.visible = false;
		this.$set(this.warningPatientList, index, item);
	  }
	  closeCard(item, index){
		if(this.ignoreSerialNo===null) return;
	    deleteWarningPatient({executeDoctorID: this.$store.state.user.token, ignoreReason: this.ignoreReason, serialNo: this.ignoreSerialNo})
		.then((response)=>{
		  //console.log(response);
		  this.success('操作成功')
		  this.doClose(item,index);
		  this.doList();
		}).catch((error)=>{
		  console.log(error)
		  this.error('操作失败')
		  this.doClose(item,index);
		  this.doList();
		});
		
	  }
	  
	  openCreateFollowRecord(patientID, alertItems){
		this.centerDialogVisible = true;
		this.selectedPatientID = patientID;
		this.selectedAlertItems = alertItems;
	  }
	  
		P_create = function(func,data){
			return new Promise((resolve, reject)=>{
				func(data)
				  .then(response=>{
					resolve(response.data);
			      })
			      .catch(err=>{
					reject(err);
			      })
			    })
		}
	  
	  submitCreateFollows(){
		var P_create_follows = this.selectedAlertItems.map((item)=>{
			var planDate = item.alertTime.replace(' ','T');
			planDate += 'Z';
			return this.P_create(createFollowRecord,{
			  alertSerialNo: item.serialNo,
			  content: this.form.content,
			  deathTime: this.form.deathTime,
			  executeDoctorID: this.$store.state.user.token,
			  failureReason: this.form.failureReason,
			  followupMethod: this.form.followupMethod,
			  followupType: this.form.followupType,
			  patientID: this.selectedPatientID,
			  planDate: planDate,
			  planSerialNo: "",
			  status: this.form.status,
			  summary: this.form.summary,
			  templateCode: 0});
		})
		Promise.all(P_create_follows)
		  .then(res=>{
			this.success('提交成功')
			this.centerDialogVisible = false;
		  })
		  .catch(err=>{
			this.error('提交失败')
		  })
	  }
	  
	  showPatientDlg(patientID){
	    this.$router.push("/patientInfo/"+patientID)
	  }
	  
	  dateToAge(str){
	    var date = new Date(str);
		var now = new Date();
		return now.getYear()-date.getYear()+1;
	  }
	  
	  doList(){
		this.isError = false;
		this.isSearching = true;
		//console.log(this.$store.state)
		getWarningPatientCount({viewerID: this.$store.state.user.token}).then(response=>{
			var patientTotal = response.data;
			getWarningPatientList({pageIndex: 1, pageOffset: Math.min(10, patientTotal), viewerID: this.$store.state.user.token}).then(response=>{
				//console.log(response);
				this.warningPatientList = response.data.content;
				this.patientTotal = patientTotal;
				this.isSearching = false;
				this.isError = false;
				console.log(this.warningPatientList);
			}).catch(error=>{
				this.isError = true;
				this.isSearching = false;
			})
		}).catch(error=>{
			this.isError = true;
			this.isSearching = false;
		})
		
		
		
	  }
	
		
		load(){
			this.isLoading=true;
			this.LoadingText="加载中";
			this.doList();
			this.isLoading=false;
			this.LoadingText="刷新";
		}
		
		created(){
		  this.$emit("activeChanged",0)
		  this.doList();
		}
	}
</script>

<style lang="scss" scoped>
.warning {
  &-container {
    margin: 30px;
  }
  &-text {
    font-size: 30px;
    line-height: 46px;
  }
}
.nameSexAge{
  font-family: '微软雅黑 Bold', '微软雅黑';
  font-weight: 700;
  font-style: normal;
  font-size: 18px;
  margin-left: 10px;
  margin-right: 40px;
}
.clearfix:before,
.clearfix:after {
  display: table;
  content: "";
}
.clearfix:after {
  clear: both
}
.WarningInfoContainer{
  display:flex;
}
.MedicationInfoContainer{
  display: flex;
}
.ManageSituationContainer{
  display: flex;
}
.blueblock{
  background-color: lightblue;
  width: 18px;
  height: 24px;
  margin-bottom: -6px;
  display: inline-block
}
.left{
  display: inline-block;
  flex:1;
  font-size: 18px;
  line-height: 40px;
}
.right{
  display: inline-block;
  flex:3;
  font-size: 18px;
  line-height: 38px;
}
.row{
  display: inline-block;
}
.alertItem{
  display: flex;
  
  .warningIcon{
    color: red;
	font-size: 20px;
	line-height: 38px;
	margin-right: 10px;
  }
  .alertNameReason{
    color: red;
	flex:1
  }
  .alertDateTime{
    color: grey;
	flex:1
  }
}
.drugItem{
  display: flex;
  .lbl{
    flex:1;
  }
  .content{
    color: grey;
	flex:1;
  }
}
.btn-ignore{
  min-width: 85px;
}
.card{
  margin:20px;
}
.footer{
  margin-top: 40px;
  text-align: center;
}
</style>
