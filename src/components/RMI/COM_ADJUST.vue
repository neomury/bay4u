<template>
    <v-app>
        <v-container class="contents">
        <v-row align="center">
             <v-col cols="12" sm="3">
                <v-autocomplete
                    class="pt-8 pr-4"
                    v-model="itemId"
                    :items="itemTypeList"
                    item-text="text"
                    item-value="value"
                    label="구분"
                    small-chips
                ></v-autocomplete>
            </v-col>   
            <v-col cols="12" sm="3">
                <v-autocomplete
                    class="pt-8 pr-4"
                    v-model="carMakerId"
                    :items="carMakerLists"
                    item-text="MakeName"
                    item-value="MakeId"
                    label="제조사"
                    small-chips
                    clearable
                    style="max-width: 200px; font-size:10px;"
                    @change="changeMakeName"
                    >
                </v-autocomplete>
            </v-col>
      </v-row>
      <v-row>
            <v-col
                cols="12"
                sm="12"
            >
                <div v-if="dataList.length > 0 && itemId === 22">
                    <span>{{dataList[0].MakeName}}</span>
                    <div v-for="(item , index) in dataList[0].CarRange" :key="index" class="adjustData">
                        <!--<div style="width:200px;">{{item.RangeName}}</div>-->
                        <div>
                            <div v-for="(item2 , index2) in item.CarTypes" :key="index2" class="carType">
                                <!--<div style="width:300px;">{{item2.TypeName}}</div>-->
                                <div>
                                    <div v-for="(item3 , index3) in item2.ItemMps" :key="index3" class="itemMps">
                                        <span>{{item2.TcdTypeId}};</span>
                                        <span>{{item2.TypeId}};</span>
                                        <span>{{item.RangeName}};</span>
                                        <span>{{item2.TypeName}};</span>
                                        <span>{{item2.engineCode}};</span>
                                        <span>{{item2.beginDate}};</span>
                                        <span>{{item2.endDate}};</span>
                                        <span>{{item3.ItemMpText}};</span>
                                        <span v-for="(item4, index4) in item3.ItemValues" :key="index4">
                                            {{item4.ValueText}}
                                        </span>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
                <div v-else-if="dataList.length > 0 && itemId === 20">
                     <span>{{dataList[0].MakeName}}</span>
                      <div v-for="(item , index) in dataList[0].CarRange" :key="index" class="adjustData">
                        <div>
                            <div v-for="(item2 , index2) in item.CarTypes" :key="index2" class="carType">
                                <div>
                                    <div v-for="(item3 , index3) in item2.SubGroups" :key="index3" >
                                        <div v-for="(item4 , index4) in item3.ItemMps" :key="index4" class="itemMps">
                                            <span v-for="(item5, index5) in item4.Values" :key="index5" class="values">
                                                <span>{{item2.TcdTypeId}};</span>
                                                <span>{{item2.TypeId}};</span>
                                                <span>{{item.RangeName}};</span>
                                                <span>{{item2.TypeName}};</span>
                                                <span>{{item2.engineCode}};</span>
                                                <span>{{item2.beginDate}};</span>
                                                <span>{{item2.endDate}};</span>
                                                <span>{{item3.SubGroupName}};</span>
                                                <span>{{item4.ItemMpText}}</span>;
                                                <span class="blank">{{item5.QualColText}};</span>{{item5.ValueText}} {{item5.QuantityText}}
                                            </span>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                      </div>
                </div>
            </v-col>
      </v-row>
        </v-container>
         <Progress v-if="isLoaded"></Progress>
    </v-app>
</template>

<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.2.1/jquery.min.js"></script>
<script>

    import Progress from '@/components/Common/Progress.vue'
    const axios = require('axios').default;
    const url = "https://rmi-services.tecalliance.net";
    const adjUrl = "https://rmi-services.tecalliance.net/rest/Adjust";
    
    export default {
        name: 'COM_ADJUST',
        data(){
            return{
                rmiAuthKey: '',
                jsonHeader: { 'Accept': 'application/json', 'Content-Type': 'application/json', 'Authorization': 'TecRMI {{AuthToken}}' },
                itemId : 0,
                itemTypeList : [{text:'소모품 교환주기', value:22},{text:'오일 교환량 상세',value:20}],
                carMakerLists: [],
                carRangeLists: [],
                carTypeLists: [],
                carMakerId: '',
                carRangeId: '',
                carTypeId: '',
                dataList : [],
                adjustList : [],
                isLoaded: false
            }
        },
        created(){
            this.getAuthKey();	
        },
        components: {
			Progress
		},
        methods:{
            getAuthKey() {
                var param = {};
                param.Company = "SK Network-South Korea_3303624";
                param.Account = "rmi-ws_004553-1-KR";
                param.Password = "SkCho123!";

                axios({
                    method: 'POST',
                    url: url + "/Auth/Login",
                    headers: this.jsonHeader,
                    data: param
                })
                .then((result) => {
                    this.rmiAuthKey = result.headers['x-authtoken'];
                    
                    console.log('rmiAuthKey :', this.rmiAuthKey);

                    // 조회 후 Maker List 조회
                    param = {};
                    param.ShowCar = true;
                    param.CountryCode = "kr";
                    param.LanguageCode = "en";

                    if(this.carVin !== undefined && this.carVin !== "") {
                        param.AddInfoKeyFilter = [{"AddInfoKeyId":-8, "AddInfoKeyFilterValue": this.carVin }];
                    }

                    this.jsonHeader.Authorization = "TecRMI " + this.rmiAuthKey;
                    console.log('rmiAuthKey : ', this.rmiAuthKey);

                    axios({
                        method: 'POST',
                        url: url + "/rest/VehicleTree/MakeList",
                        headers: this.jsonHeader,
                        data: param
                    })
                    .then((result) => {
                        console.log('Maker List 조회 : ', result);
                        this.carMakerLists = result.data;                   
                    });    
                });            
            },
            changeMakeName() {
                console.log('changeMakeId : ', this.carMakerId);
                
                this.dataList.push(this.carMakerLists.find(x =>x.MakeId === this.carMakerId));
                
                this.carRangeLists = [];
                this.carTypeLists = [];

                this.carRangeId = '';
                this.carTypeId = '';

                if(this.carMakerId !== undefined) {
                    var param = {};
                    param.ShowCar = true;
                    param.MakeId = this.carMakerId;
                    param.CountryCode = "kr";
                    param.LanguageCode = "en";

                    if(this.carVin !== undefined && this.carVin !== "") {
                        param.AddInfoKeyFilter = [{"AddInfoKeyId":-8, "AddInfoKeyFilterValue": this.carVin }];
                    }              

                    this.jsonHeader.Authorization = "TecRMI " + this.rmiAuthKey;

                    axios({
                        method: 'POST',
                        url: url + "/rest/VehicleTree/RangeList",
                        headers: this.jsonHeader,
                        data: param
                    })
                    .then((result) => {
                        this.carRangeLists = result.data;
                        var carRangeList = this.carRangeLists.map(function(item){
                            var obj = {};
                            obj.RangeId = item.RangeId;
                            obj.RangeName = item.RangeName;
                            obj.CarTypes = [];
                            return obj;
                        });
                        console.log('RangeList 조회 : ', this.carRangeLists);

                        this.dataList = this.dataList.map(function(item2){
                            var obj2 = {};
                            obj2.MakeId = item2.MakeId;
                            obj2.MakeName = item2.MakeName;
                            obj2.CarRange = carRangeList;
                            return obj2;
                        });

                        console.log('dataList : ', this.dataList);
                        this.changeRangeName();
                    });       
                }      
            },
            async changeRangeName() {
                //console.log('changeRangeName : ', this.carRangeId);

                this.carTypeLists = [];
                this.carTypeId = '';
                this.isLoaded = true;
                for(var element of this.dataList ){
                    
                    for (var subElement of element.CarRange){
                        //console.log('subElement :',subElement.RangeId);
                        
                        var param = {};
                        param.ShowCar = true;
                        param.RangeId = subElement.RangeId;
                        param.CountryCode = "kr";
                        param.LanguageCode = "en";

                        this.jsonHeader.Authorization = "TecRMI " + this.rmiAuthKey;
                        
                        await axios({
                            method: 'POST',
                            url: url + "/rest/VehicleTree/TypeListTcd",
                            headers: this.jsonHeader,
                            data: param
                        })
                        .then((result) => {
                            this.carTypeLists = result.data;

                            // 차량 상세 정보를 추가로 넣어 줌. (엔진코드, 연식, 마력 등)
                            this.carTypeLists = this.carTypeLists.map(obj=>{
                                var carTypeItem = {};
                                var carTypeId = {"TypeId" : obj.TypeId, "TcdTypeId" : obj.TcdTypeId};
                                carTypeItem.TypeId = obj.TypeId;
                                carTypeItem.TcdTypeId = obj.TcdTypeId;

                                var beginDate = obj.TypeDetails.filter(x => x.AddInfoKeyId === 3)[0].AddInfoKeyValue;
                                beginDate = beginDate.substring(0,4) +'-'+ beginDate.substring(4);
                                
                                var endDateList = obj.TypeDetails.filter(x => x.AddInfoKeyId === 4);
                                var endDate = '';
                                if(endDateList.length === 1){
                                    endDate = endDateList[0].AddInfoKeyValue;
                                    endDate = endDate.substring(0,4) +'-'+ endDate.substring(4);
                                }
                                
                                var enginCodeList = obj.TypeDetails.filter(x => x.AddInfoKeyId === -4);
                                var engineCode = '';
                                if (enginCodeList.length > 0) {
                                    //engineCode = ' ( ';

                                    for(let i=0; i<enginCodeList.length; i++) {
                                        engineCode = engineCode + enginCodeList[i].AddInfoKeyValue;
                                        if(i !== enginCodeList.length-1) engineCode += ', ';
                                    }

                                    //engineCode += ' ) ';
                                }

                                // 미터마력(ps) = 키로와트(kw) * 1.36
                                var kw =  obj.TypeDetails.filter(x => x.AddInfoKeyId === 5)[0].AddInfoKeyValue;
                                var ps = Math.round(kw * 1.36);
                                
                                carTypeItem.TypeName = '\'' + obj.TypeName; //+  engineCode  + ' [ ' + beginDate + '~' + endDate + ' ] '+ ps +' hp';
                                carTypeItem.engineCode = engineCode; 
                                carTypeItem.beginDate = '\'' + beginDate + '\''; 
                                carTypeItem.endDate = endDate;
                                carTypeItem.hp = ps +' hp'; 

                                if(this.itemId === 22){
                                    carTypeItem.ItemMps = [];
                                }
                                else{
                                    carTypeItem.SubGroups = [];
                                }
                                
                                return carTypeItem;

                            });

                            //console.log('TypeList 조회 : ', this.carTypeLists);

                            subElement.CarTypes = this.carTypeLists;

                        });
                    };
                };
            
                var self = this;
                console.log('itemtype :' , this.itemId);
                if(this.itemId === 22){
                   
                    this.getChangeData(function(data){
                        console.log('data :', data);
                        self.adjustList = data;
                        self.isLoaded = false;
                    });
                }
                else{
                    this.getOilData(function(data){
                        console.log('data :', data);
                        self.adjustList = data;
                        self.isLoaded = false;
                    });
                }
                this.$forceUpdate();
            },
            getChangeData(callback){

                //console.log('dataList : ', this.dataList.length);
                
                this.dataList.forEach(element => {
                   
                    element.CarRange.forEach(carRangeEl => {
                        //console.log('carRangeEl : ', carRangeEl);
                 
                        carRangeEl.CarTypes.forEach(carTypeEl => {
                          //console.log('carTypeEl : ', carTypeEl);
                 
                           let languageCode = 'en',
                                countryCode = 'kr',
                                typeId = carTypeEl.TypeId,
                                subGroupId = '95';		
                                
                            // Build url query string
                            let query = '?subGroupId='+ subGroupId
                                + '&countryCode=' + countryCode
                                + '&languageCode=' + languageCode
                                + '&typeId=' + typeId	
                            
                            // Send HTTP request
                            let xmlHttp = new XMLHttpRequest();
                            xmlHttp.open( 'GET', adjUrl + '/SubGroupData' + query, false );
                            xmlHttp.setRequestHeader( 'Content-type', 'application/json;charset=UTF-8' );
                            xmlHttp.setRequestHeader( 'Accept', 'application/json' );
                            xmlHttp.setRequestHeader( 'Authorization', "TecRMI " + this.rmiAuthKey);
                            xmlHttp.send( null );
                            
                            // Handle HTTP response
                            if(xmlHttp.status == 200) {
                                //console.log('changeMainGroup 리턴 : ', JSON.parse(xmlHttp.responseText));
                                var result = JSON.parse(xmlHttp.responseText);
                                if(result !== null)
                                {
                                    carTypeEl.ItemMps = result.ItemMps.map(function(item){
                                        var obj = {};
                                        obj.ItemMpText = item.ItemMpText;
                                        obj.ItemValues = item.Values;
                                        return obj;
                                    });
                                }
                            }
                        });
                    });
                
                });

                callback(this.dataList);
            },
            getOilData(callback){
                console.log('getOilData..... ');

                this.dataList.forEach(element => {
                   
                    element.CarRange.forEach(carRangeEl => {
                        //console.log('carRangeEl : ', carRangeEl);
                 
                        carRangeEl.CarTypes.forEach(carTypeEl => {
                          //console.log('carTypeEl : ', carTypeEl);
                 
                           let  languageCode = 'en',
                                countryCode = 'kr',
                                typeId = carTypeEl.TypeId,
                                mainGroupId = '20';		
                                
                            // Build url query string
                            let query = '?mainGroupId='+ mainGroupId
                                + '&countryCode=' + countryCode
                                + '&languageCode=' + languageCode
                                + '&typeId=' + typeId	
                            
                            // Send HTTP request
                            let xmlHttp = new XMLHttpRequest();
                            xmlHttp.open( 'GET', adjUrl + '/MainGroupData' + query, false );
                            xmlHttp.setRequestHeader( 'Content-type', 'application/json;charset=UTF-8' );
                            xmlHttp.setRequestHeader( 'Accept', 'application/json' );
                            xmlHttp.setRequestHeader( 'Authorization', "TecRMI " + this.rmiAuthKey);
                            xmlHttp.send( null );
                            
                            // Handle HTTP response
                            if(xmlHttp.status == 200) {
                                
                                var result = JSON.parse(xmlHttp.responseText);
                                if(result !== null)
                                {
                                    //console.log(result);
                                    carTypeEl.SubGroups = result.map(function(item){
                                        var obj = {};
                                        obj.SubGroupId = item.SubGroupId;
                                        obj.SubGroupName = item.SubGroupName;
                                        obj.ItemMps = item.ItemMps.map(function(item2){
                                            var obj2 = {};
                                            obj2.ItemMpText = item2.ItemMpText;
                                            obj2.ItemMpId = item2.ItemMpId;
                                            obj2.Values = item2.Values;
                                            return obj2;
                                        });
                                        return obj;
                                    });
                                }
                            }
                        });
                    });
                
                });

                callback(this.dataList);
            },
        }
    }
</script>
<style scoped>
    .contents{
        font-size: 0.6em;
    }
    .adjustData{
        display:flex;
        border-bottom: 1px solid gray;
    }
    .adjustData .carType{
        display: flex;
        border-bottom: 1px solid gray;
    }
    .adjustData .carType :last-child{
        border: 0px;
    }
    .itemMps{
       display: block;
       padding:3px;
    }
    .itemMps :last-child{
        border: 0px;
    }
    .itemMps span{
       margin-right:20px;
    }
    .blank{
        margin-left:10px;
    }
    .values{
        display: block;
    }
</style>