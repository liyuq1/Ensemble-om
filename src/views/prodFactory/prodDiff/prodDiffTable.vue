<template>
  <div>
    <v-card md20 lg20 ref="print">
      <v-card-text v-show="RB">
        <v-tabs fixed-tabs grow show-arrows slot="extension" v-if="isTable == false">
          <v-tab v-for="n in diffList" :key="n" class="diffTitle">{{n}}</v-tab>
          <v-tabs-items v-model="model">
            <v-tab-item v-for="i in diffList" :key="i">
              <prod-diff v-if="i=='产品属性'" :prodData="prodDefineData"></prod-diff>
              <prod-diff v-if="i=='开户定义'" :prodData="prodEventOpen"></prod-diff>
              <prod-diff v-if="i=='销户定义'" :prodData="prodEventClose"></prod-diff>
              <prod-diff v-if="i=='存入定义'" :prodData="prodEventCret"></prod-diff>
              <prod-diff v-if="i=='支取定义'" :prodData="prodEventDebt"></prod-diff>
              <base-table v-if="i=='收费定义'" :prodCharge="prodCharge"></base-table>
              <base-table v-if="i=='利率定义'" :prodCharge="prodInt"></base-table>
              <base-table v-if="i=='核算定义'" :prodCharge="prodAccounting"></base-table>
            </v-tab-item>
          </v-tabs-items>
        </v-tabs>
      </v-card-text>
      <v-card-text v-show="CL">
        <v-tabs fixed-tabs grow show-arrows slot="extension" v-if="isTable == false">
          <v-tab v-for="n in diffList2" :key="n" class="diffTitle">{{n}}</v-tab>
          <v-tabs-items v-model="model">
            <v-tab-item v-for="i in diffList2" :key="i">
              <prod-diff v-if="i=='产品属性'" :prodData="prodDefineData"></prod-diff>
              <prod-diff v-if="i=='开户定义'" :prodData="prodEventOpen"></prod-diff>
              <prod-diff v-if="i=='放款定义'" :prodData="prodEventDrw"></prod-diff>
              <prod-diff v-if="i=='还款定义'" :prodData="prodEventRec"></prod-diff>
              <prod-diff v-if="i=='到期信息'" :prodData="prodEventDue"></prod-diff>
              <base-table v-if="i=='利率信息'" :prodCharge="prodInt"></base-table>
              <base-table v-if="i=='变更信息'" :prodCharge="prodAmend"></base-table>
              <base-table v-if="i=='核算信息'" :prodCharge="prodAccounting"></base-table>
            </v-tab-item>
          </v-tabs-items>
        </v-tabs>
      </v-card-text>

      <v-card-text v-show="GL">
        <v-tabs fixed-tabs grow show-arrows slot="extension" v-if="isTable == false">
          <v-tab v-for="n in diffListGl" :key="n" class="diffTitle">{{n}}</v-tab>
          <v-tabs-items v-model="model">
            <v-tab-item v-for="i in diffListGl" :key="i">
              <prod-diff v-if="i=='产品属性'" :prodData="prodDefineData"></prod-diff>
              <prod-diff v-if="i=='开户定义'" :prodData="prodEventOpen"></prod-diff>
              <prod-diff v-if="i=='销户定义'" :prodData="prodEventClose"></prod-diff>
              <prod-diff v-if="i=='贷记事件'" :prodData="prodEventCret"></prod-diff>
              <prod-diff v-if="i=='借记事件'" :prodData="prodEventDebt"></prod-diff>
              <base-table v-if="i=='利率信息'" :prodData="prodInt"></base-table>
              <base-table v-if="i=='核算信息'" :prodCharge="prodAccounting"></base-table>
            </v-tab-item>
          </v-tabs-items>
        </v-tabs>
      </v-card-text>

      <v-card-text v-if="isTable == true">
        <v-tabs fixed-tabs v-if="isTable == true">
          <v-tab style="margin-left: 0px">{{diffTitles}}</v-tab>
          <v-tabs-items>
            <v-tab-item>
              <base-table :prodCharge="prodCharge"></base-table>
            </v-tab-item>
          </v-tabs-items>
        </v-tabs>
      </v-card-text>
    </v-card>
  </div>
</template>

<script>

    import prodDiff from '@/views/prodFactory/prodDiff/prodDiff'
    import baseTable from '@/views/prodFactory/prodInfo/table/baseTable'
    import {getParamTable} from "@/api/url/prodInfo";
    import { getDiffList } from "@/api/url/prodInfo";
    import { getDiffTable } from "@/api/url/prodInfo";
    import {PrintInfo} from '@/utils/print/print'
    import { getAttrType} from "@/api/url/prodInfo";
    import { getModuleByFlowCode } from "@/api/url/prodInfo";
    import DcTextField from '@/components/widgets/DcTextField'
    import TaskListFlex from '@/views/propertyManage/taskListFlex'
    import { getProdData } from "@/api/prod";
    import download2 from '@/utils/download2';
    import toast from '@/utils/toast';
    import {
        tranFlowInfo
    } from '@/api/url/prodInfo';
    import {
        tranFlowRelease
    } from '@/api/url/prodInfo';
export default {
    components: {
        prodDiff,
        baseTable,
        TaskListFlex,
        DcTextField
    },
    props: ['prodData','sendTranId','sendTranType','seqNo'],

    data (){
        return {
            sourceModule: [],
            spinning: false,
            RB: false,
            CL: false,
            GL: false,
            releaseFlowInfo: [],
            temp: [],
            title: "",
            diffTitles: "",
            checkFlowInfo: [],
            confirmInfo: [],
            code: '',
            optKey: 0,
            showFlag: 0,
            optValue: '',
            optDesc: '',
            prodData: {},
            isTable: false,
            prodCharge: {},
            prodInt: {},
            prodDefineData: {},
            prodEventOpen: {},
            prodEventClose: {},
            prodEventCret: {},
            prodEventCycle: {},
            prodEventDebt: {},
            diffList: ["产品属性","开户定义","销户定义","存入定义","支取定义","收费定义","利率定义","核算定义"],
            prodEventDrw: {},
            prodEventRec: {},
            prodEventDur: {},
            prodAccounting: {},
            prodAmend: {},
            diffList2: ["产品属性","开户定义","放款定义","还款定义","到期信息","利率信息","变更信息","核算定义"],
            diffListGl: ["产品属性","开户定义","销户定义","借记事件","贷记事件","利率信息","核算定义"],

            prodGroup: [{
                key: 'Y',
                value: 'Y-是'
            }],
            prodClass: [{
                key: 'RB100',
                value: 'RB100-A'
            }],
            prodRange: [{
                key: '',
                value: ''
            }],
            status: [{
                key: '',
                value: ''
            }],
            baseProdType: [{
                key: '',
                value: ''
            }],
            prodType: [{
                key: '',
                value: ''
            }],
            tranId: '',
            attrType: "",
        }
    },
  computed: {
    randomColor() {
      let item = Math.floor(Math.random() * this.colors.length);
      return this.colors[item];
    }
  },
  watch: {
      sendTranId: {
          handler(val,oldVal)
          {
              if(val!=oldVal){
                  this.$nextTick(() => {
                      this.changeTranId(this.sendTranId,this.sendTranType)
                  })
              }
          }
      },
      seqNo: {
          handler(val,oldVal)
          {
              if(val!=oldVal){
                  this.$nextTick(() => {
                      this.changeTranId(this.sendTranId,this.sendTranType)
                  })
              }
          }
      },
   },
  methods: {
      printDown () {
          PrintInfo(this.$refs.print)
      },
      changeTranId(tranId,tranType){
          this.RB = false;
          this.CL = false;
          this.GL = false;
          this.isTable = false
          if(tranType=='0'){
              this.isTable = false
              this.getDiffProdData(tranId);
          }else{
              this.isTable = true
              this.getDiffTableData(tranId);
          }
      },
      //通过交易主单号 获取单表差异信息
      getDiffTableData(tranId) {
          this.attrType = getAttrType()
          let heads = []
          var data={'mainSeqNo': this.seqNo,"tranId": tranId};
          getDiffTable(data).then(response => {
              console.log(response);
              this.diffTitles = tranId
              heads = response.data.data.columnHeard;
              let tableDiffInfo = response.data.data.tableInfo
              //获取单表列描述
              let heards=[];
              let assembleColumns=[];
              for(let i=0; i<heads.length; i++){
                  let head={};
                  head["text"]=heads[i].title
                  head["value"]=heads[i].code
                  heards.push(head);
              }
              //区分数据操作类型
              let opt={};
              opt["text"]="操作类型";
              opt["value"]= "dmlType";
              heards.push(opt);

              //组装差异数据
              for(let i in tableDiffInfo){
                  //新增参数
                  let optTypeDesc = ""
                  if(tableDiffInfo[i].dmlType === "I"){
                      optTypeDesc = "新增"
                  }
                  if(tableDiffInfo[i].dmlType === "D"){
                      optTypeDesc = "删除"
                  }
                  if(tableDiffInfo[i].dmlType === "U"){
                      optTypeDesc = "修改"
                  }
                  if(tableDiffInfo[i].dmlType == 'I'){
                      tableDiffInfo[i].newData["dmlType"] = optTypeDesc
                      assembleColumns.push(tableDiffInfo[i].newData)
                  }else if(tableDiffInfo[i].dmlType == 'D'){
                      //删除数据
                      tableDiffInfo[i].oldData["dmlType"] = optTypeDesc
                      assembleColumns.push(tableDiffInfo[i].oldData)
                  }else if(tableDiffInfo[i].dmlType == 'U'){
                      //修改数据
                      for (let col in tableDiffInfo[i].newData) {
                          let newCol = tableDiffInfo[i].newData[col]
                          let oldCol = tableDiffInfo[i].oldData[col]
                          if(newCol !== oldCol){
                              tableDiffInfo[i].newData[col] = oldCol + ">" + newCol
                          }
                      }
                      tableDiffInfo[i].newData["dmlType"] = optTypeDesc
                      assembleColumns.push(tableDiffInfo[i].newData)
                  }
              }
              const column = {"headers": heards,"column": assembleColumns}
              this.prodCharge= column;
          });
      },
      //通过交易主单号 获取产品差异信息
      getDiffProdData(tranId){
          let data={'mainSeqNo': this.seqNo,'tranId': tranId};
          getModuleByFlowCode(this.seqNo).then(response => {
              for(let tId in response.data.data){
                  if(tranId == tId){
                      this.sourceModule = response.data.data[tId].SOURCE_MODULE
                  }
              }
              getDiffList(data).then(response => {
                  this.prodData = response.data.data;
                  if (this.sourceModule == "RB") {
                      this.RB = true
                      //产品属性
                      this.assembleProdDefine();
                      //存款的开户定义，销户定义，存入定义，支取定义
                      this.assembleEvent();
                      //将收费定义的差异组装
                      this.assembleProdCharge();
                      //将利率定义的差异组装
                      this.assembleProdInt();
                      //将核算定义的差异组装
                      this.assembleAccounting();
                  }
                  if (this.sourceModule == "CL") {
                      this.CL = true
                      //产品属性
                      this.assembleProdDefine();
                      //贷款的开户定义，放款定义，还款定义，到期信息
                      this.assembleCLEvent();
                      //将收费定义的差异组装
                      this.assembleProdCharge();
                      //将变更定义的差异组装
                      this.assembleAmend();
                      //将核算定义的差异组装
                      this.assembleAccounting();
                  }
                  if (this.sourceModule == "GL") {
                      this.GL = true
                      //产品属性
                      this.assembleProdDefine();
                      //内部账的开户定义,销户定义,借记事件，贷记事件
                      this.assembleEvent();
                      //将利率定义的差异组装
                      this.assembleProdInt();
                      //将核算定义的差异组装
                      this.assembleAccounting();
                  }
                  this.prodGroup = this.prodData.mbProdType.prodGroup
                  this.prodClass = this.prodData.mbProdType.prodClass
                  this.prodDesc = this.prodData.mbProdType.prodDesc
                  this.prodType = this.prodData.mbProdType.prodGroup
                  this.status = this.prodData.mbProdType.status
                  this.baseProdType = this.prodData.mbProdType.baseProdType
                  this.prodRange = this.prodData.mbProdType.prodRange
                  this.prodType = this.prodData.mbProdType.prodType
              });
          })
      },
      //产品属性差异
      assembleProdDefine() {
          const prodDefine=this.prodData.prodDefine;
          const prodDefineDiff=this.prodData.diff.prodDefine;
          const prodType=this.prodData.prodType;
          const baseEffectProd = this.prodData.baseEffectProd;
          const prodDefineData={"prodDefines": prodDefine,"diff": prodDefineDiff,"prodType": prodType,"baseEffectProd": baseEffectProd};
          this.prodDefineData=prodDefineData;
      },
      assembleEvent(){
          const prodEvent=this.prodData.prodEvent;
          const prodEventDiff=this.prodData.diff.mbProdEvent;
          const prodType=this.prodData.prodType;
          const baseEffectProd = this.prodData.baseEffectProd;
          //区分差异
          const openDiff={}
          const closeDiff={}
          const cycleDiff={}
          const debtDiff={}
          const cretDiff={}
          for(const diffKey in prodEventDiff){
              const key=diffKey.substring(diffKey.indexOf('.')+1);
              if(diffKey.indexOf('OPEN')>=0){
                  openDiff[key]=prodEventDiff[diffKey];
              }
              if(diffKey.indexOf('CLOSE')>=0){
                  closeDiff[key]=prodEventDiff[diffKey];
              }
              if(diffKey.indexOf('CRET')>=0){
                  cretDiff[key]=prodEventDiff[diffKey];
              }
              if(diffKey.indexOf('DEBT')>=0){
                  debtDiff[key]=prodEventDiff[diffKey];
              }
          }
          for(const keyD in prodEvent){
              const openEvent={"prodDefines": prodEvent[keyD],"prodType": prodType}
              if(keyD.indexOf('OPEN')>=0){
                  openEvent["diff"]=openDiff
                  openEvent["baseEffectProd"]=baseEffectProd
                  this.prodEventOpen=openEvent;
              }else if(keyD.indexOf('CLOSE')>=0){
                  openEvent["diff"]=closeDiff
                  openEvent["baseEffectProd"]=baseEffectProd
                  this.prodEventClose= openEvent
              }else if(keyD.indexOf('CRET')>=0){
                  openEvent["diff"]=cretDiff
                  openEvent["baseEffectProd"]=baseEffectProd
                  this.prodEventCret= openEvent
              }
              else if(keyD.indexOf('DEBT')>=0){
                  openEvent["diff"]=debtDiff
                  openEvent["baseEffectProd"]=baseEffectProd
                  this.prodEventDebt= openEvent
              }
          }
          if(JSON.stringify(prodEvent)=='{}' || JSON.stringify(prodEvent) == undefined){
              let diffEvent={"prodType": prodType}
              let diffEventClose={"prodType": prodType}
              let diffEventCret={"prodType": prodType}
              let diffEventDebt={"prodType": prodType}
              diffEvent["diff"]=openDiff
              this.prodEventOpen=diffEvent;
              diffEventClose["diff"]=closeDiff
              this.prodEventClose= diffEventClose;
              diffEventCret["diff"]=cretDiff
              this.prodEventCret= diffEventCret;
              diffEventDebt["diff"]=debtDiff
              this.prodEventDebt= diffEventDebt;
          }
      },
      //收费定义差异组装
      assembleProdCharge(){
          const prodInfo=this.prodData.mbProdCharge;
          const prodChargeDiff=this.prodData.diff.mbProdCharge;
          let assembleColumns=[];
          let heards=[];
          let type = "";
          if(prodInfo == undefined){
              type = "copy"
          }else if(prodInfo.length != 0){
              type = "change"
          }
          if(type == "change") {
              for (const key in prodInfo[0]) {
                  let head = {};
                  head["text"] = this.getAttrDesc(key);
                  head["value"] = key;
                  heards.push(head);
              }
              let head = {};
              head["text"] = "操作类型";
              head["value"] = "dmlType";
              heards.push(head);
              for (const prodCharge in prodInfo) {
                  const chargeColumn = prodInfo[prodCharge];
                  const keyAndValue = "{\"FEE_TYPE\":\"" + chargeColumn.feeType + "\",\"PROD_TYPE\":\"" +
                      chargeColumn.prodType + "\"}";
                  const diff = prodChargeDiff[keyAndValue];
                  if (diff == undefined) {
                      assembleColumns.push(chargeColumn)
                  } else {
                      for (const col in chargeColumn) {
                          let chargeCol = chargeColumn[col];
                          let diffCol = diff[col];
                          if (chargeCol != diffCol) {
                              chargeColumn[col] = chargeCol + '>' + diffCol
                          }
                      }
                      chargeColumn["dmlType"] = "修改"
                      assembleColumns.push(chargeColumn)
                  }
              }
          }
          if(type != "change"){
              for(const child in prodChargeDiff){
                  for(const key in prodChargeDiff[child]){
                      let head={};
                      head["text"]=this.getAttrDesc(key);
                      head["value"]=key;
                      heards.push(head);
                  }
                  break;
              }
              for(const index in prodChargeDiff){
                  const prodCharge= prodChargeDiff[index];
                  const dmlType=prodCharge.dmlType;
                  let diffData={}
                  if(dmlType == 'I'){
                      for(const num in heards){
                          const value=heards[num].value
                          if(value == "dmlType"){
                              diffData[value] = "新增"
                              continue
                          }
                          diffData[value]= prodCharge[value];
                      }
                      assembleColumns.push(diffData)
                  }
              }
          }
          const reColumn = {"headers": heards,"column": assembleColumns}
          this.prodCharge= reColumn;
      },
      //利率定义差异组装
      assembleProdInt(){
          const prodInfo=this.prodData.irlProdInt;
          const prodChargeDiff=this.prodData.diff.irlProdInt;
          let assembleColumns=[];
          let heards=[];
          let type = "";
          if(prodInfo == undefined){
              type = "copy"
          }else if(prodInfo.length != 0){
              type = "change"
          }
          if(type == "change") {
              for (const key in prodInfo[0]) {
                  let head = {};
                  head["text"] = this.getAttrDesc(key);
                  head["value"] = key;
                  heards.push(head);
              }
              let head = {};
              head["text"] = "操作类型";
              head["value"] = "dmlType";
              heards.push(head);
              for (const prodCharge in prodInfo) {
                  const chargeColumn = prodInfo[prodCharge];
                  const keyAndValue = "{\"SPLIT_ID\":\"" + chargeColumn.splitId +
                      "\",\"INT_CLASS\":\"" + chargeColumn.intClass +
                      "\",\"EVENT_TYPE\":\"" + chargeColumn.eventType +
                      "\",\"PROD_TYPE\":\"" + chargeColumn.prodType +
                      "\",\"RULEID\":\"" + chargeColumn.ruleid + "\"}";
                  const diff = prodChargeDiff[keyAndValue];
                  if (diff == undefined) {
                      assembleColumns.push(chargeColumn)
                  } else {
                      for (const col in chargeColumn) {
                          let chargeCol = chargeColumn[col];
                          let diffCol = diff[col];
                          if (chargeCol != diffCol) {
                              chargeColumn[col] = chargeCol + '>' + diffCol
                          }
                      }
                      chargeColumn["dmlType"] = "修改"
                      assembleColumns.push(chargeColumn)
                  }
              }
          }
          if(type != "change"){
              for(const child in prodChargeDiff){
                  for(const key in prodChargeDiff[child]){
                      let head={};
                      head["text"]=this.getAttrDesc(key);
                      head["value"]=key;
                      heards.push(head);
                  }
                  break;
              }
              for(const index in prodChargeDiff){
                  const prodCharge= prodChargeDiff[index];
                  const dmlType=prodCharge.dmlType;
                  let diffData={}
                  if(dmlType == 'I'){
                      for(const num in heards){
                          const value=heards[num].value
                          if(value == "dmlType"){
                              diffData[value] = "新增"
                              continue
                          }
                          diffData[value]= prodCharge[value];
                      }
                      assembleColumns.push(diffData)
                  }
              }
          }

          const reColumn = {"headers": heards,"column": assembleColumns}
          this.prodInt= reColumn;
      },
      //核算定义的差异组装
      assembleAccounting(){
          const prodInfo=this.prodData.glProdAccounting;
          const prodChargeDiff=this.prodData.diff.glProdAccounting;
          let assembleColumns=[];
          let heards=[];
          let type = "";
          if(prodInfo == undefined){
              type = "copy"
          }else if(prodInfo.length != 0){
              type = "change"
          }
          if(type == "change") {
              for (const key in prodInfo[0]) {
                  let head = {};
                  head["text"] = this.getAttrDesc(key);
                  head["value"] = key;
                  heards.push(head);
              }
              let head = {};
              head["text"] = "操作类型";
              head["value"] = "dmlType";
              heards.push(head);
              for (const prodCharge in prodInfo) {
                  const chargeColumn = prodInfo[prodCharge];
                  const keyAndValue = "{\"PROD_TYPE\":\"" + chargeColumn.prodType + "\",\"ACCOUNTING_STATUS\":\"" +
                      chargeColumn.accountingStatus + "\"}";
                  const diff = prodChargeDiff[keyAndValue];
                  if (diff == undefined) {
                      assembleColumns.push(chargeColumn)
                  } else {
                      for (const col in chargeColumn) {
                          let chargeCol = chargeColumn[col];
                          let diffCol = diff[col];
                          if (chargeCol != diffCol) {
                              chargeColumn[col] = chargeCol + '>' + diffCol
                          }
                      }
                      chargeColumn["dmlType"] = "修改"
                      assembleColumns.push(chargeColumn)
                  }
              }
          }
          if(type != "change"){
              for(const child in prodChargeDiff){
                  for(const key in prodChargeDiff[child]){
                      let head={};
                      head["text"]=this.getAttrDesc(key);
                      head["value"]=key;
                      heards.push(head);
                  }
                  break
              }
              for(const index in prodChargeDiff){
                  const prodCharge= prodChargeDiff[index];
                  const dmlType=prodCharge.dmlType;
                  let diffData={}
                  if(dmlType == 'I'){
                      for(const num in heards){
                          const value=heards[num].value
                          if(value == "dmlType"){
                              diffData[value] = "新增"
                              continue
                          }
                          diffData[value]= prodCharge[value];
                      }
                      assembleColumns.push(diffData)
                  }
              }
          }
          const reColumn = {"headers": heards,"column": assembleColumns}
          this.prodAccounting= reColumn;
      },
      //变更定义的差异组装
      assembleAmend(){
          const prodInfo=this.prodData.mbProdAmendMaping;
          const prodChargeDiff=this.prodData.diff.mbProdAmendMaping;
          let assembleColumns=[];
          let heards=[];
          let type = "";
          if(prodInfo == undefined){
              type = "copy"
          }else if(prodInfo.length != 0){
              type = "change"
          }
          if(type == "change") {
              for (const key in prodInfo[0]) {
                  let head = {};
                  head["text"] = this.getAttrDesc(key);
                  head["value"] = key;
                  heards.push(head);
              }
              let head = {};
              head["text"] = "操作类型";
              head["value"] = "dmlType";
              heards.push(head);
              for (const prodCharge in prodInfo) {
                  const chargeColumn = prodInfo[prodCharge];
                  const keyAndValue = "{\"PROD_TYPE\":\"" + chargeColumn.prodType + "\"}";
                  const diff = prodChargeDiff[keyAndValue];
                  if (diff == undefined) {
                      assembleColumns.push(chargeColumn)
                  } else {
                      for (const col in chargeColumn) {
                          let chargeCol = chargeColumn[col];
                          let diffCol = diff[col];
                          if (chargeCol != diffCol) {
                              chargeColumn[col] = chargeCol + '>' + diffCol
                          }
                      }
                      chargeColumn["dmlType"] = "修改"
                      assembleColumns.push(chargeColumn)
                  }
              }
          }
          if(type != "change"){
              for(const child in prodChargeDiff){
                  for(const key in prodChargeDiff[child]){
                      let head={};
                      head["text"]=this.getAttrDesc(key);
                      head["value"]=key;
                      heards.push(head);
                  }
                  break
              }
              for(const index in prodChargeDiff){
                  const prodCharge= prodChargeDiff[index];
                  const dmlType=prodCharge.dmlType;
                  let diffData={}
                  if(dmlType == 'I'){
                      for(const num in heards){
                          const value=heards[num].value
                          if(value == "dmlType"){
                              diffData[value] = "新增"
                              continue
                          }
                          diffData[value]=prodCharge[value];
                      }
                      assembleColumns.push(diffData)
                  }
              }
          }
          const reColumn = {"headers": heards,"column": assembleColumns}
          this.prodAmend= reColumn;
      },

      assembleCLEvent(){
          const prodEvent=this.prodData.prodEvent;
          const prodEventDiff=this.prodData.diff.mbProdEvent;
          const prodType=this.prodData.prodType;
          const baseEffectProd = this.prodData.baseEffectProd;
          //区分差异
          const cycleDiff={}
          const openDiff={}
          const drwDiff={}
          const recDiff={}
          const dueDiff={}
          for(const diffKey in prodEventDiff){
              const key=diffKey.substring(diffKey.indexOf('.')+1);
              if(diffKey.indexOf('OPEN')>=0){
                  openDiff[key]=prodEventDiff[diffKey];
              }
              if(diffKey.indexOf('DRW')>=0){
                  drwDiff[key]=prodEventDiff[diffKey];
              }
              if(diffKey.indexOf('REC')>=0){
                  recDiff[key]=prodEventDiff[diffKey];
              }
              if(diffKey.indexOf('DUE')>=0){
                  dueDiff[key]=prodEventDiff[diffKey];
              }
          }
          for(const key in prodEvent){
              const openEvent={"prodDefines": prodEvent[key],"prodType": prodType}
              if(key.indexOf('OPEN')>=0){
                  openEvent["diff"]=openDiff
                  openEvent["baseEffectProd"]=baseEffectProd
                  this.prodEventOpen=openEvent;
              }
              else if(key.indexOf('DRW')>=0){
                  openEvent["diff"]=drwDiff
                  openEvent["baseEffectProd"]=baseEffectProd
                  this.prodEventDrw= openEvent
              }else if(key.indexOf('REC')>=0){
                  openEvent["diff"]=recDiff
                  openEvent["baseEffectProd"]=baseEffectProd
                  this.prodEventRec= openEvent
              }else if(key.indexOf('DUE')>=0){
                  openEvent["diff"]=dueDiff
                  openEvent["baseEffectProd"]=baseEffectProd
                  this.prodEventDue= openEvent
              }
          }
          if(JSON.stringify(prodEvent)=='{}' || JSON.stringify(prodEvent) == undefined){
              let diffEvent={"prodType": prodType}
              let diffEventDrw={"prodType": prodType}
              let diffEventRec={"prodType": prodType}
              let diffEventDue={"prodType": prodType}
              diffEvent["diff"]=openDiff
              this.prodEventOpen=diffEvent;
              diffEventDrw["diff"]=drwDiff
              this.prodEventDrw= diffEventDrw
              diffEventRec["diff"]=recDiff
              this.prodEventRec= diffEventRec
              diffEventDue["diff"]=dueDiff
              this.prodEventDue= diffEventDue
          }
      },

      getAttrDesc(val){
          let desc = ""
          if(/[a-z]/.test(val)){
              val = val.replace(/([A-Z])/g,"_$1").toUpperCase();
          }
          for(let i=0; i<this.attrType.length; i++){
              if(val == this.attrType[i].attrKey){
                  desc = this.attrType[i].attrDesc
                  break
              }
          }
          if(desc == ""){
              desc = val
          }
          return desc
      }

  }
};
</script>
<style scoped>
  .diffTitle {
    font-size: large;
  }
</style>