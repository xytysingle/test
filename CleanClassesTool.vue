<template>
    <div>
        <!-- 展示切图列表的大图 -->
        <div ref="showSIBI" id="showSIBI">
            <img :src="nowSIUrl" >
        </div>
        <!-- 创建选框 -->
        <div ref='selectDiv' id="selectDiv">

        </div>

        <!-- 创建分类弹窗 -->
        <div ref="createNewClassLayer" id="createNewClassLayer" >
            <p id="title">创建分类</p>
            <div class="el-icon-close" @click="closeCreateNewClassLayer()">

            </div>
            <div class="create-nc-input">
               <el-input id="newClassInput" v-model="newFolder" placeholder="请输入内容" clearable :autofocus="true"></el-input>
            </div>
            <div id="newClassNameTips">
                <div v-for="(item, index) in NCskusfilter" :key="index">
                    <div class="item-row" @click="completeNewClassName(item)">
                        <p class="item-row-name">{{item.sku_name}}</p>
                    </div>
                    
                </div>   
            </div>
            <div id="newClassBtn">
                <el-button @click="createnewclass()" type="info" plain style="width:400px">确定</el-button>
            </div>
        </div>
        <!-- 显示大图 -->
        <div id="bigImage" ref="bigImage" >
            <img ref='bigImagePhoto' id="bigImagePhoto" :src="thisIconUrl" :onerror="errorImg">
        </div>

        <div class="clean-left">
            <!-- 原图 -->
            <div class="clean-middle-top">
                <div id="clean-original-image" >
                    
                    <!-- <img src="http://ubuntu.zhixiang.co:8889/index.php/image/view?file=801101.jpg" alt=""> -->
                    <canvas ref="imagelager" id="image-layer"></canvas>
                    <canvas ref="showrectlayer" id="show-rect-layer"></canvas>
                    
                </div>
                         
                
            </div>
            
            <div id="clean-control">
                    <!-- 测试button -->
                    <!-- <div id="testButton">
                        <el-button type="primary" @click="controlScroll">主要按钮</el-button>
                    </div> -->
                    <div id="totalClassCount">
                        <p>总分类数: {{classList.length}}</p>
                    </div>
                    <div id="smallImagesCount">
                        <p>当前页切图数: {{slicedClassimagesList.length}}</p>
                    </div>
                    <div id="wqxClassCount">
                        <p>未清洗: {{wqxCount}}</p>
                    </div>
                    <div id="yqxClassCount">
                        <p>已清洗: {{yqxCount}}</p>
                    </div>
                    <div id="dqrClassCount">
                        <p>待确认: {{dqrCount}}</p>
                    </div>
                    <div id="cleansectionNumber">
                        <!-- <p>清洗区间: {{cleansectionStartNum}}-{{cleansectionEndNum}}</p> -->
                        <p>切图名称: {{choosedSlicedImageName}}</p>
                        
                        
                    </div>
                    <div id="smallTobigImg">
                        <img :src="nowImgUrl" alt="">
                    </div>
                    <div id="createnewclass">
                        <el-button @click="getCreateNewClassLayer()" plain>新建分类</el-button>
                    </div>
                    

                    <!-- 原图放大缩小 -->
                    <div id="originalImageBigSmall" ref="originalImageBigSmall">
                        <div id="bigOriginalImage" class="el-icon-zoom-in" @click="bigOriginalImage()">
                           
                        </div>
                        <div id="smallOriginalImage" class="el-icon-zoom-out" @click="smallOriginalImage()">
                            
                        </div>
                    </div>

                    <div id="big-small-smallPhotos">
                        
                        <div id="big-smallPhotos" class="el-icon-zoom-in" @click="bigsmallPhotos()">
                           
                        </div>
                        <div id="small-smallPhotos" class="el-icon-zoom-out" @click="smallsmallPhotos()">
                            
                        </div>
                        
                    </div>
            </div>

            <div class="search-filtrate">
                <div class="search-class">
                    <el-input
                        placeholder="请输入内容"
                        prefix-icon="el-icon-search"
                        v-model="inputkeywords">
                    </el-input>
                </div>
                <div class="filtrate-class">
                    <el-select v-model="value" placeholder="筛选条件" @change="filtrateClass()">
                        <el-option
                        v-for="item in options"
                        :key="item.value"
                        :label="item.label"
                        :value="item.value">
                        </el-option>
                    </el-select>
                </div>

                
            </div>

            <!-- 清洗列表 -->
            <div class="clean-class-list">
                <div class="clean-class-item" :class="index==nowChoosedClassIndex?'choosedStyle':''" v-for="(item, index) in skusfilter" :key="index">
                    <div id="class-image" @mouseenter="showBigImage(item, index, $event)" @mouseleave="hideBigImage()" @click="addToTargetClass(item)" >
                        <!-- <img :src=item.url :onerror="errorImg" @click="addToTargetClass(item)"> -->
                        <el-image :src=item.url fit="contain"   lazy></el-image>
                    </div>
                    <div id="class-name">
                        <p>{{item.name}}</p>
                    </div>
                    <div id="class-flag">
                        <div >
                            <el-radio-group v-model="item.state" size="mini" @change="changeState(item, index)">
                                <el-radio-button label="未清洗" ></el-radio-button>
                                <el-radio-button label="已清洗"></el-radio-button>
                                <el-radio-button label="待确认"></el-radio-button>
                            </el-radio-group>
                        </div>
                    </div>
                    <div id="classListClick" @click="updateOneClassAndshowSmallImages(item, index)">
                        
                    </div>
                </div>
            </div>
        </div>
        <div id="dragArea" @mousemove="moveChoosedSI($event)"  @mouseup="changeMouseDownToUp($event)"> <!--  -->
            <!-- <div class="clean-middle" id="clean-middle"> -->
            
                <!-- fabric实现清洗区域 -->
                <!-- <canvas id="fabric-clean-area" width="800px" height="800px"></canvas> -->
                <!-- 小图列表 -->
                <!-- @mousedown="mousedown($event)" @mousemove="mousemove($event)" @mouseup="mouseup($event)" -->
                <div @contextmenu="preventDefaultMenu()" class="clean-classimages-area" id="cleanClassimagesArea" @click="hideSearchingPanel()"> 
                    <!-- <div id="smallImagesGroup" draggable="true">
                    
                    </div> -->

                    <!-- <div  ref="slicedClassimageItem" @keyup.37="goToLeftItem()" @keyup.39="goToRightItem()" @mousedown="hideSIBigImage()" @click.exact="getOriginalImage()" @click.shift="selectContinuously()" draggable="true" class="slicedClassimageItem" :class="item.isChoosed?'smallImageChoosed':''"  @dragstart="drag($event)"> -->
                    <div  ref="slicedClassimageItem" @contextmenu="showMenu(item, index, $event)" @mouseenter="showSIBigImage(item, $event)" @click.alt="selectGap(item, index)"  @mousedown="setMouseDownPosition($event, item, index)"   @keyup.37="goToLeftItem(index)" @keyup.39="goToRightItem(index)"  @click.exact="getOriginalImage(item, index)" @click.shift="selectContinuously(item, index)" draggable="false" class="slicedClassimageItem" :class="item.isChoosed?'smallImageChoosed':''" :id="gernerateId(item)" v-for="(item, index) in slicedClassimagesList" :key="index">
                        <img @contextmenu="showMenu(item, index, $event)" :src="item.url" alt="" draggable="false"  @mouseenter="showSIBigImage(item, $event)" > <!--@mouseleave="hideSIBigImage()"-->
                        <!-- <div id="smallImgItem">

                        </div>
                        <el-image :src=item.url fit="contain"   lazy></el-image> -->
                        <p draggable="false" id="smallImageName">{{item.name | imgIndexFilter}}</p>
                    </div>

                    
                            <vue-context-menu 
                                :contextMenuData="contextMenuData"
                                @showMenuDelete="showMenuDelete()"
                                style="background:black;"
                            ></vue-context-menu>

                </div>

                <!-- <canvas id="cleanArea" width="800px" height="800px"></canvas> -->
            <!-- </div> -->
            <div class="clean-right" id="cleanRightdiv">
                <div id="tradeFilter">
                    <el-select v-model="tradeID" filterable placeholder="请选择" @change="filtrateClassByChoosedTrade()">
                        <el-option
                        v-for="item in tradeList"
                        :key="item.value"
                        :label="item.label"
                        :value="item.value">
                        </el-option>
                    </el-select>
                </div>
                <div id="targetClassSearchRefresh">
                    <el-button id="targetClassSearchRefreshBtn" type="primary" @click="getAllClasses()">刷新分类</el-button>
                </div>
                <div id="searchingPanel">
                    <div id="searchingPanelIterm" v-for="(item, index) in NCskusfilter" :key="index" @click="chooseNowTargetItem(item, index)">
                        <p>{{item.sku_name}}</p>
                    </div>
                </div>
                <div id="targetClassSearch">
                    <div id="targetClassSearchInput" >
                        <el-input
                        placeholder="请输入内容"
                        prefix-icon="el-icon-search"
                        v-model="newClassName"
                        @focus="getSearchingPanel()" 
                        clearable
                        >
                        
                        </el-input>
                    </div>
                    
                     
                </div>
                <div id="nowTargetItemDropArea">
                    <el-button @click="addToTargetClass(nowTargetItem)" id="addItemToBtn" type="warning" icon="el-icon-star-off" circle></el-button>
                    <img :src="nowTargetItem.url" :alt="nowTargetItem.sku_name">
                </div>
                <div id="targetScrollArea">
                    <div id="classAcceptArea" class="class-accept-area" v-for="(item, index) in TargetClassList" :key="index">
                        <div class="el-icon-error" @click="removeTargetClass(index)">

                        </div>
                        <!-- <div class="class-accept-image">
                            
                            <img :src=item.url :onerror="errorImg">
                        </div> -->
                        <div id="class-accept-name">
                                <p>{{item.name}}</p>
                        </div>
                        <div id="cleanRight" draggable="false" class="class-accept-rect" >
                            <img :src=item.url :onerror="errorImg" draggable="false">
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <div id="pagination">
            <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="currentPage"
            :page-sizes="[100, 200, 300, 400]"
            :page-size="100"
            layout="total, sizes, prev, pager, next, jumper"
            :total="totalSmallImagesCount">
            </el-pagination>
        </div>
    </div>
</template>


<script>
    let dom = null;
    import '@/assets/css/cleanclassestool.css'
    export default {
        data() {
            return {
                getAllClassesUrl:"index.php/skus/get-all",
                isClickaple: true,
                searchClass: '',
                options: [{
                    value: '全部',
                    label: '全部'
                    }, {
                    value: '已清洗',
                    label: '已清洗'
                    }, {
                    value: '未清洗',
                    label: '未清洗'
                    }, {
                    value: '待确认',
                    label: '待确认'
                    }],
                value: '全部',
                radiochoose: 1,
                cleansectionStartNum: 100001,
                cleansectionEndNum: 200001,
                choosedSlicedImageName: '746263_322_336_328_344_0',
                slicedClassimagesList: [],
                testItemName: '746568_717_1125_941_1201_0',
                getimgurl: 'index.php/image/view?',
                getannourl: '/index.php/annotation/view',
                taskImgUrl: '',
                classList: [],
                TargetClassList: [],
                sizeNumber: 1,
                spSizeNumber: 1.0,
                bboxes: [],
                nowCleanImgIndex: 9,
                nowItem: {},
                nowIndex: 0,
                annoTop: 0,
                annoLeft: 0,
                skulisturl: '/index.php/skus/list?type=6',
                skusList: [],
                errorImg:'this.src="' + require('@/assets/img/errorimg.png') + '"',
                // errorImg: require('@/assets/img/errorimg.png'),
                thisIconUrl: '',
                nowChoosedClassIndex: 9999,
                inputkeywords: '',
                nowresults: {},
                newClassName: '',
                remainclass: [],
                classState: [],
                // 鼠标框选参数
                startX: 0,
                startY: 0,
                endX: 0,
                endY: 0,
                
                nowChoosedSmallImageIndex: 999999,
                nowSIUrl: '',
                areaTop: 0,
                contextMenuData: {
                    menuName: '属性修改',
                    axis: {
                        x: null,
                        y: null
                    },
                    menulists: [
                        {
                        fnHandler: 'showMenuDelete',
                        icoName: 'el-icon-delete',
                        btnName: '删除图片'
                        },
                    ]
                },
                isShowMenu: false,
                nowDeleteItem:{},
                isResetCleanState: false,
                poolId: "",
                choosedSmallImageGroup: [],
                canvas: '',
                ismouseDownT: false,
                mouseDownX: null,
                mouseDownY: null,
                mouseUpX: null,
                mouseUpY: null,
                targetTop: 0,
                targetLeft: 0,
                isHaveCopy: false,
                wqxCount: 0,
                yqxCount: 0,
                dqrCount: 0,
                nowImgUrl: '',
                isDraged: false,
                tradeID: "26",
                isAltKeyDown: false,
                isShiftKeyDown: false,
                tradeList: [],
                allClasses: [],
                nowTargetItem: {},
                newFolder: '',
                currentPage: 1,
                totalSmallImages: [],
                totalSmallImagesCount: 0,
                pageSizeCount: 100,
                








            }
        },

        computed:{
            
            NCskusfilter: function() {
                let skus = this.remainclass;
                let keywords = this.newClassName;
                let key = keywords.trim().split(/\s+/);
                let keylength = key.length;
                let skusResult = [];
                if(keylength > 0){
                    for(let j=0; j<keylength; j++){
                        for(let i=0; i< skus.length; i++){
                            if(skus[i].sku_name.search(key[j]) != -1){
                                skusResult.push(skus[i]);
                            }
                        }
                        skus = skusResult;
                        skusResult = [];    
                    }
                        
                }

                return skus
            },
            
            skusfilter: function(){
                // console.log('=================')
                let skus = this.classList;
                let keywords = this.inputkeywords;
                let key = keywords.trim().split(/\s+/);
                let keylength = key.length;
                let skusResult = [];
                if(keylength > 0){
                    for(let j=0; j<keylength; j++){
                        for(let i=0; i< skus.length; i++){
                            if(skus[i].name.search(key[j]) != -1){
                                skusResult.push(skus[i]);
                            }
                        }
                        skus = skusResult;
                        skusResult = [];    
                    }
                        
                }
                // 添加筛选过滤
                if(this.value != '全部'){
                    for(let i=0; i<skus.length; i++){
                        if(this.value == skus[i].state) {
                            skusResult.push(skus[i])
                        }
                    }
                    skus = skusResult;
                    skusResult = [];
                }

                return skus
            },
        },
        watch:{
            spSizeNumber: {
                handler: function (val, oldVal) { 
                    // Math.round(num * 100) / 100
                    // let size = Math.round(this.sizeNumber * 100) / 100;
                    // console.log(size);
                    console.log('老：', oldVal);
                     console.log('新：', val);
                     console.log(this.spSizeNumber);
                     this.bigOrSmallSmallPhotos();
                     
                },
                deep: true
                
            },
            sizeNumber: {
                handler: function (val, oldVal) { 
                    // Math.round(num * 100) / 100
                    // let size = Math.round(this.sizeNumber * 100) / 100;
                    // console.log(size);
                    console.log('老：', oldVal);
                     console.log('新：', val);
                     console.log(this.sizeNumber);
                    //  this.createAnnotationArea();
                    this.createImageLayer(this.nowresults);
                    this.ShowRect(this.nowresults);
                     
                },
                deep: true
                
            },
            currentPage: {
                handler: function (val, oldVal) { 
                    // Math.round(num * 100) / 100
                    // let size = Math.round(this.sizeNumber * 100) / 100;
                    // console.log(size);
                    console.log('老：', oldVal);
                     console.log('新：', val);
                    this.pagingSmallImages();
                     
                },
                deep: true
                
            },
            pageSizeCount: {
                handler: function (val, oldVal) { 
                    // Math.round(num * 100) / 100
                    // let size = Math.round(this.sizeNumber * 100) / 100;
                    // console.log(size);
                    console.log('老：', oldVal);
                     console.log('新：', val);
                    this.pagingSmallImages();
                     
                },
                deep: true
                
            },
        },
        filters: {
            imgIndexFilter(results) {
                results = results.split('_');
                return results[0]
            }
        },
        created(){
            // document.onkeydown = function(e) {
            //     console.log("keydown:",e);
            // }
           
            
            // document.onkeyup = function(event){
            //     console.log("keyup:",event);
            // }
            // 获取行业列表
            this.getTradeList();

            console.log('created:', this.$route.params.typeValue);
            

            this.skulisturl = '/index.php/skus/list?type=' + this.$route.params.typeValue;
            this.isResetCleanState = this.$route.params.isResetCleanState;
            this.poolId = this.$route.params.poolId;

            this.resetLocalStorge();

            // 储存行业id
            if(this.$route.params.typeValue){
                localStorage.setItem("tradeID", JSON.stringify(this.$route.params.typeValue))
                localStorage.setItem("poolId", JSON.stringify(this.$route.params.poolId))
            }else {
                this.getTradeIDFromLocalStorge();
            }
            

            // 获取可标注的全部分类和相应logo图
            this.Axios.get(
                    // `${this.api}/index.php/skus/list?type=${this.tradeID}`,
                    `${this.api}/${this.getAllClassesUrl}`,
                    // `${this.api}${this.skulisturl}`,
                    {params: {
                            'token':this.token
                        }
                    }
                    
                )
                .then(response =>{
                    //   console.log(response)
                    if(response.status==200){
                        this.skusList = response.data;

                         for(let i=0; i<this.skusList.length; i++ ){
                  
                            if(this.skusList[i].url){
                                // console.log(this.skusList[i].url);
                                
                                this.skusList[i].url = `${this.api}` + this.skusList[i].url.slice(1)
                                this.skusList[i].name = this.skusList[i].sku_name;
                            }else {
                                this.skusList[i].url = require('@/assets/img/errorimg.png');
                                this.skusList[i].name = this.skusList[i].sku_name;
                            }
                            
                            
                            // let skusObj = {
                            //     id: this.skusList[i].id,
                            //     sku_name: this.skusList[i].sku_name,
                            //     type_id: this.skusList[i].type_id,
                            //     //   url: 'http://ubuntu.zhixiang.co:8889' + this.skusList[i].url,
                            //     url: this.skusList[i].url,
                            //     // color: this.color[i % 10]
                            // }
                            // //   console.log(this.skusList[i].id, this.skusList[i].sku_name, this.skusList[i].type_id, this.skusList[i].url, this.color[i % 10]);
                            // //   console.log(skusObj);
                            // this.skusList[i] = skusObj;
                            
                        }
                        this.allClasses = this.skusList;
                        this.filtrateClassByChoosedTrade();

                        // console.log('获取的skusList:',this.skusList);


            // 如果是清洗酒水饮料，提供关联分类的数据list
            // if(this.$route.params.typeValue == 1){
            //     console.log("饮料酒水")
            //     // 获取啤酒类列表
            //     this.getpinleiList(11)
            //     this.getpinleiList(7)
            //     this.getpinleiList(23)
            //     this.getpinleiList(18)
                
            //     setTimeout(()=>{
            //         this.getTargetClass()
            //     }, 0)
            // }else{
            //     this.getTargetClass()
            // }
            setTimeout(()=>{
                    this.getTargetClass()
                }, 0)
                



            

                        // console.log('skusList为', response.data.data.skus)
                        // 获取待清洗的分类
            
                
            

            }
        })
        .catch(error => {
                console.log(error)
            });


            
            // setTimeout(()=>{
            //     this.getRemainClass();
            // },0)
            

        },
        mounted(){
             
             // 获取滚动条滚动坐标
            window.addEventListener('scroll', this.handleScroll, true)

            // 监听小图展示区的scroll
            window.addEventListener('scroll', this.smallImageHandleScroll, true)

            window.addEventListener('scroll', this.handleTargetAreaScroll, true)
            
            // this.createSelectRect();
            
            //监听键盘事件
            document.onkeyup = function(event){
            // console.log('-----------------')
                let _key = window.event.keyCode;
                let _index = this.nowChoosedSmallImageIndex;
                // console.log(_key, _index);
                if(_key == 37){
                    console.log('调用了left');
                this.goToLeftItem(_index);
                }else if(_key == 39){
                    console.log('调用了right');
                    this.goToRightItem(_index);
                }
                this.isAltKeyDown = false;
                this.isShiftKeyDown = false;
                
            
            }.bind(this);

            document.onkeydown = function(event){
            // console.log('-----------------')
                let _key = window.event.keyCode;
                this.isAltKeyDown = true;
                this.isShiftKeyDown = true;
                console.log(_key);
                
                
            
            }.bind(this);

            
            // this.createCleanArea();
            
        },

        methods: {
            cleanAreaScrollToZeros() {
                document.getElementById("cleanClassimagesArea").scrollTop = 0;
            },
            pagingSmallImages() {
                let allPagesCount = parseInt(this.totalSmallImagesCount / this.pageSizeCount) + 1;
                if(this.currentPage < allPagesCount){
                    this.slicedClassimagesList = this.totalSmallImages.slice(this.pageSizeCount * (this.currentPage -1), this.pageSizeCount * this.currentPage)
                }else if(this.currentPage == allPagesCount){
                    this.slicedClassimagesList = this.totalSmallImages.slice(this.pageSizeCount * (this.currentPage -1), this.totalSmallImagesCount)
                }
                console.log("目前页码的LIST：",this.slicedClassimagesList)
                
            },
            handleSizeChange(val) {
                console.log(`每页 ${val} 条`);
                this.pageSizeCount = val;
            },
            handleCurrentChange(val) {
                this.cleanAreaScrollToZeros();
                this.currentPage = val;
                console.log(`当前页: ${val}`);
                console.log(`当前页2: ${this.currentPage}`);
            },
            getSearchingPanel() {
                console.log("获得焦点")
                document.getElementById("searchingPanel").style.display = "block";
            },
            hideSearchingPanel() {
                console.log()
                console.log("失去焦点")
                document.getElementById("searchingPanel").style.display = "none";
            },
            chooseNowTargetItem(item, index) {
                this.newClassName = item.name;
                this.hideSearchingPanel();
                this.nowTargetItem = item;
                let flag = false;
                let tempObject = {};
                let params = {};
                params.name = this.nowTargetItem.sku_name;
                params.dir = this.poolId;
                for(let i=0; i<this.classList.length; i++) {
                    if(this.newClassName == this.classList[i].name) {
                        flag = true;
                        tempObject = this.classList[i];
                        break;
                    }  
                }
                if(flag) {
                    // for(let i = 0; i<this.TargetClassList.length; i++){
                    //     if(this.TargetClassList[i].name == params.name) {
                    //         this.TargetClassList.splice(i, 1);
                    //         break;
                    //     }
                    // }
                    // this.TargetClassList.splice(0, 0, tempObject);
                    // this.newClassName = '';
                }else {
                    this.Axios.post(
                        // `${this.api}/index.php/annotation/view`,
                        `${this.api}/index.php/classimages/mk`,
                        params
                            
                        
                            // params
                        )
                        .then(response =>{
                            // console.log(response);
                            if(response.data.code == 200) {
                                let tempObject = {};
                                tempObject.name = params.name;
                                tempObject.photos = [];
                                tempObject.state = "未清洗";
                                tempObject.url = this.getLogoByName(params.name)
                                // this.classList.splice(0, 0, tempObject);
                                this.classList.push(tempObject);
            

                                // this.TargetClassList.splice(0, 0, tempObject);
                                // this.newClassName = '';
                            }
                        })
                        .catch(error => {
                            console.log(error);
                        })
                }
            },
            filtrateClassByChoosedTrade() {

                console.log("this.tradeID:", this.tradeID)
                if(parseInt(this.tradeID) == 26) {
                    this.remainclass = this.allClasses;
                }else {
                    this.remainclass = []
                    for(let item of this.allClasses) {
                        if(item.type_id == this.tradeID) {
                            this.remainclass.push(item)
                        }

                    }
                }
            },
            getTradeList() {
                this.Axios.get(
                    `${this.api}/index.php/skus/type-list`,
                    // params
                )
                .then(response =>{
                    // console.log(this.loginAccount, this.loginPassword);
                    console.log(response.data);
                    if(response.data.code == 200){
                        // console.log('---------')
                        let tradeType = response.data.data.sku_type;
                        for(let item of tradeType){
                            item.label = item.type_name;
                            item.value = item.id;
                            this.tradeList.push(item)
                        }
                        
                        console.log()    

                    }
                })
                .catch(error =>{
                    console.log(error);
                })
            },
            // 滚动右侧到顶部
            scrollToTopRight(){
                document.getElementById("targetScrollArea").scrollTop = 0;
                // this.targetTop = "0px";
            },
            
            getTradeIDFromLocalStorge() {
                this.tradeID =JSON.parse(localStorage.getItem('tradeID')) ;
                this.poolId =JSON.parse(localStorage.getItem('poolId')) ;
            },
            
            copyChoosedSI() {
                if(this.choosedSmallImageGroup.length > 0){
                    for(let item of this.choosedSmallImageGroup) {
                        let areaW = window.getComputedStyle(document.getElementById("cleanClassimagesArea")).width
                        let div = document.getElementById(item.id)
                        let box = div.cloneNode(true)
                        // let divX = window.getComputedStyle(div).top;
                        let divW = window.getComputedStyle(div).width;
                        let divH = window.getComputedStyle(div).width;
                        box.style.position = "absolute"
                        
                        box.style.top = parseInt(item.index / parseInt(parseInt(areaW) / (parseInt(divW) + 30)))  * (parseInt(divH) + 30) - this.areaTop + "px";
                        // box.style.top = document.getElementById(item.id).getBoundingClientRect().top;
                        box.style.left = parseInt(item.index % parseInt(parseInt(areaW) / (parseInt(divW) + 30)))  * (parseInt(divW) + 30) + "px";
                        // box.style.left = document.getElementById(item.id).getBoundingClientRect().left;
                        // box.style.left = item.index % parseInt(areaW / (divW + 30)) * (divH + 30) + "px";
                        // console.log("box.style.top",parseInt(areaW) / (parseInt(divW) + 30))
                        box.id = item.id + "_copy"
                        console.log("divW:", divW, "divH:", divH, "areaW:", areaW, "this.areaTop:",this.areaTop)
                        document.getElementById("dragArea").appendChild(box)
                    }
                }
                
            },
            changeToTargetClass(event) {
                let endMX = event.clientX; 
                let endMY = event.clientY; 
                this.isDraged = false;
                let _targetLeft = parseInt(window.getComputedStyle(document.getElementById("cleanRightdiv")).left) + parseInt(window.getComputedStyle(document.getElementById("clean-original-image")).width)
                console.log("nowTargetItem.length:",this.nowTargetItem)
                let smallImageNameList = []
                for(let item of this.choosedSmallImageGroup) {
                    smallImageNameList.push(item.name)
                }

                if(endMX > _targetLeft && endMY < 330 && this.nowTargetItem.name){
                        console.log("搜索放置区")
                        // for(let item of this.choosedSmallImageGroup) {
                            // console.log(this.nowTargetItem)
                            // setTimeout(()=>{
                        this.changeClass(smallImageNameList, this.choosedSmallImageGroup[0].className, this.nowTargetItem)
                            // }, 0)

                            
                        // }
                    }else{
                        for(let i=0; i< this.TargetClassList.length; i++) {
                        
                        let _targetTop = i * 300 - this.targetTop
                        // console.log("targetLeft",_targetLeft, _targetTop, this.targetTop)
                        // console.log("endMX:",endMX, endMY)
                            if(endMX > _targetLeft && _targetTop < endMY && endMY < _targetTop + 300) {
                                // for(let item of this.choosedSmallImageGroup) {
                                //     // console.log(this.TargetClassList[i])
                                    
                                //     setTimeout(()=>{
                                this.changeClass(smallImageNameList,this.choosedSmallImageGroup[0].className, this.TargetClassList[i])
                                    // }, 0)

                                    
                                // }
                            }
                        }
                    }
                
            },
            handleScroll () {
            // let scrollTop = window.pageYOffset || document.documentElement.scrollTop || document.body.scrollTop
            // console.log(scrollTop)
        //    let cmsY = this.$refs.contentmiddle.offsetTop;
            if(document.getElementById("anno-canvas")){
                this.annoTop = document.getElementById("anno-canvas").getBoundingClientRect().top;
                this.annoLeft = document.getElementById("anno-canvas").getBoundingClientRect().left;
            }
           
                // console.log("this.annoTop:",this.annoTop)
                // console.log("this.annoLeft:",this.annoLeft)
                
        },
            
            // canvas清洗
            // createCleanArea() {
            //     let canvasW = document.getElementById("clean-middle").offsetWidth;
            //     let canvasH = document.getElementById("clean-middle").offsetHeight;
            //     console.log("canvasW:", canvasW)
            //     this.canvas = new fabric.Canvas('cleanArea', {
            //         hoverCursor: 'pointer',
            //         width: canvasW,
            //         height: canvasH,

            //     })
            // },
            
            // shift连续选择
            selectContinuously(item, index){
                this.choosedSmallImageGroup = []
                for(let _item of this.slicedClassimagesList) {
                    _item.isChoosed = false;
                }
                if(this.nowChoosedSmallImageIndex < index){
                    for(let i=this.nowChoosedSmallImageIndex; i<=index; i++) {

                        this.slicedClassimagesList[i].isChoosed = true;
                        this.slicedClassimagesList[i].index = i;
                        this.choosedSmallImageGroup.push(this.slicedClassimagesList[i])

                    }
                    
                }else{
                    for(let i=index; i<=this.nowChoosedSmallImageIndex; i++) {

                        this.slicedClassimagesList[i].isChoosed = true;
                        this.slicedClassimagesList[i].index = i;
                        this.choosedSmallImageGroup.push(this.slicedClassimagesList[i])

                    }
                }

                console.log("this.choosedSmallImageGroup:", this.choosedSmallImageGroup)
    
            },
            selectGap(item, index) {
                // this.isAltKeyDown = true;
                // console.log(item, index)
                if(this.slicedClassimagesList[index].isChoosed) {
                    this.slicedClassimagesList[index].isChoosed = false;
                    for(let i=0; i<this.choosedSmallImageGroup.length; i++) {
                        if(this.choosedSmallImageGroup[i].id == item.id) {
                            this.choosedSmallImageGroup.splice(i, 1)
                            break;
                        }

                    }
                }else{
                    item.isChoosed = true;
                    item.index = index;
                    this.choosedSmallImageGroup.push(item)
                }
                
                
            },


            // 被选中的切图添加到一个可以拖动的dom中
            addChoosedSIToDraggableDom() {
                let groupDom = document.getElementById("smallImagesGroup")
                groupDom.style.backgroundColor = "rgba(0, 0, 0, 0.8)"
                
                for(let item of this.choosedSmallImageGroup) {
                    // console.log("-item-:", item)
                    let smallImageElement = document.getElementById(item.id)
                    groupDom.appendChild(smallImageElement)
                }
            },

            // 获取啤酒分类
            getpinleiList(_id){
                this.Axios.get(
                    // `${this.api}/index.php/skus/list?type=1`,
                    `${this.api}/index.php/skus/list?type=${_id}`,
                    {params: {
                            'token':this.token
                        }
                    }
                    
                )
                .then(response =>{
                    if(response.data.code==200){
                        let _skulist = response.data.data.skus
                        console.log("_skulist:", _skulist)
                        for(let i=0; i<_skulist.length; i++ ){
                            if(_skulist[i].url){
                                // console.log(_skulist[i].url);
                                _skulist[i].url = 'http://annotation.lingmou.ai:8000' + _skulist[i].url.slice(1)
                            }
                            
                            
                            let skusObj = {
                                id: _skulist[i].id,
                                sku_name: _skulist[i].sku_name,
                                type_id: _skulist[i].type_id,
                                //   url: 'http://ubuntu.zhixiang.co:8889' + this.skusList[i].url,
                                url: _skulist[i].url,
                                // color: this.color[i % 10]
                            }
                            //   console.log(this.skusList[i].id, this.skusList[i].sku_name, this.skusList[i].type_id, this.skusList[i].url, this.color[i % 10]);
                            //   console.log(skusObj);
                            this.skusList.push(skusObj)

                        }
                        // console.log('追加后skusList:',this.skusList);
                    }
                    
                    
                })
                .catch(error => {
                    console.log(error);
                })

            },

            // 获取要清洗的分类
            getTargetClass(){
                let params = {};
                params.dir = this.poolId;
                console.log("dir:", this.poolId)
                this.Axios.post(
            // `${this.api}/index.php/annotation/view`,
            `${this.api}/index.php/classimages/list`,
            params
                    
                
                // params
            )
            .then(response =>{
                // console.log("+++++++++++++++:", response);
                if(response.data.code == 200) {
                    
                    let resData = response.data.data;
                    // console.log(resData);
                    for(let key in resData) {
                        let tempObject = {};
                        tempObject.name = key;
                        tempObject.photos = resData[key];
                        tempObject.state = this.getLocalStorge(key) ;
                        tempObject.url = this.getLogoByName(key)
                        // console.log(key, ':', resData[key]);
                        this.classList.push(tempObject)
                    }    
                }
            })
            .catch(error => {
                console.log(error);
            })
            // console.log("classList:",this.classList);

            },

            //鼠标控制左右移动
            goToLeftItem(index){
                // console.log('左',index);
                if(index > 0){
                    let leftIndex = index - 1;
                    let leftItem = this.slicedClassimagesList[leftIndex];
                    this.nowChoosedSmallImageIndex = leftIndex;
                    this.nowImgUrl = this.slicedClassimagesList[leftIndex].url;
                    this.getOriginalImage(leftItem, leftIndex);
                }
                
            },
             goToRightItem(index){
                //  console.log('右',index);
                if(index < this.slicedClassimagesList.length - 1){
                    let rightIndex = index + 1;
                    let rightItem = this.slicedClassimagesList[rightIndex];
                    this.nowImgUrl = this.slicedClassimagesList[rightIndex].url;
                    this.nowChoosedSmallImageIndex = rightIndex;
                    this.getOriginalImage(rightItem, rightIndex);
                }
                
            },

            //右键菜单部分
            preventDefaultMenu(){
                event.preventDefault();
            },
            showMenu(item, index, event) {
                event.preventDefault();
                this.nowChoosedSmallImageIndex = index;
                this.nowDeleteItem = item;
                // console.log('item:',item);
                this.isShowMenu = true;

                
                let x = event.clientX;
                let y = event.clientY;
                this.contextMenuData.axis = {
                x, y
                }
            },
            notShowMenu(){
                event.preventDefault();
            },
             showMenuDelete(){
                this.isShowMenu = false;
                // console.log('删除图片操作！')
                // console.log(this.nowDeleteItem);
                // for(let item of this.choosedSmallImageGroup) {
                this.deleteSmallImage();
                // }
                
                // this.deleteResultClass(this.nowMenuIndex)
            },
            //删除切图
            deleteSmallImage(item){
                
                let smallImageNameList = []
                for(let item of this.choosedSmallImageGroup) {
                    smallImageNameList.push(item.name)
                }
                let params ={};
                params.old = this.choosedSmallImageGroup[0].className;
                params.new = '';
                params.filename = smallImageNameList;
                params.dir = this.poolId;
                
                // console.log('old:',params.old ,'new:',params.new ,'filename:',params.filename);
                this.Axios.post(
                        // `${this.api}/index.php/annotation/view`,
                        `${this.api}/index.php/classimages/mv`,
                        params
                        
                    )
                    .then(response =>{
                        // console.log(response);
                        if(response.data.code == 200) {
                            this.$message({
                            message: '删除成功！',
                            type: 'success'
                            });
                            for(let item of this.choosedSmallImageGroup){
                                for(let i=0; i<this.slicedClassimagesList.length; i++) {
                                    if(this.slicedClassimagesList[i].id == item.id) {
                                        
                                        this.slicedClassimagesList.splice(i, 1);
                                        break;
                                    }
                                }
                            }
                            
                           
                            // this.updateOneClassAndshowSmallImages(this.nowItem, this.nowIndex);

                            // setTimeout(()=>{
                            //     // document.getElementById(this.gernerateId(data.item)).style.background = 'red';
                            // document.getElementById(this.gernerateId(item)).style.display = 'none';
                            //     // document.getElementById('cleanClassimagesArea').appendChild(document.getElementById(this.gernerateId(data.item)));
                            //     // document.getElementById('cleanRight').removeChild(document.getElementById(this.gernerateId(data.item)));

                            // },0);
                            
                        }else{
                            this.errorAlert(response.data.msg);
                            // setTimeout(()=>{
                            //     // document.getElementById(this.gernerateId(data.item)).style.background = 'red';
                            //     document.getElementById(this.gernerateId(item)).style.display = 'block';
                            //     // document.getElementById('cleanClassimagesArea').appendChild(document.getElementById(this.gernerateId(data.item)));
                            //     // document.getElementById('cleanRight').removeChild(document.getElementById(this.gernerateId(data.item)));

                            // },10);
                        }
                    })
                    .catch(error => {
                        console.log(error);
                        this.errorAlert(error.response.data.msg);
                    })

            },
            //获取小图列表的展示区的top
            smallImageHandleScroll() {
            
            if(document.getElementById("cleanClassimagesArea")){
                // this.areaTop = document.getElementById("cleanClassimagesArea").getBoundingClientRect().top;
                this.areaTop = document.getElementById("cleanClassimagesArea").scrollTop;
                // this.areaTop = document.getElementById("cleanClassimagesArea").scrollTop;
                // this.annoLeft = document.getElementById("cleanClassimagesArea").getBoundingClientRect().left;
                // console.log('areaTop:', this.areaTop);
            }
                
            },
            //展示小图的大图
            showSIBigImage(item, event){
                this.nowImgUrl = item.url;
                // this.$refs.showSIBI.style.display = "block";
                // // console.log(item)
                // this.nowSIUrl = item.url;
                // this.mouseX = event.clientX;
                // this.mouseY = event.clientY;
                // let SIY = document.getElementById(item.id).getBoundingClientRect().top;
                // // let SIY = this.areaTop;
                // let SIX = document.getElementById(item.id).getBoundingClientRect().left;
                // // let SIX = document.getElementById(item.id).getBoundingClientRect().left;
                // let SIW = parseInt(window.getComputedStyle(document.getElementById(item.id)).width);
                // console.log('SIX:',SIX,'SIY:',SIY, 'SIW:',SIW, item.id);
                // this.$refs.showSIBI.style.top = (SIY - 5) + "px";
                // this.$refs.showSIBI.style.left = (SIX - 400 + SIW/2) + "px";

            },
            hideSIBigImage(event){
                this.$refs.showSIBI.style.display = "none";
                // this.ismouseDown = true;
                // this.mouseDownX = event.clientX;
                // this.mouseDownY = event.clientY;
                // console.log("mouseDownPos:", this.mouseDownX, event.clientY)
            },
            setMouseDownPosition(event, item, index){
                let flag = false;
                for(let _item of this.choosedSmallImageGroup) {
                    if(_item.id == item.id) {
                        flag = true
                    }
                }
                if(flag) {
                    this.$refs.showSIBI.style.display = "none";
                    this.ismouseDownT = true;
                    this.mouseDownX = event.clientX;
                    this.mouseDownY = event.clientY;
                    // console.log("mouseDownPos:", this.mouseDownX, event.clientY, this.ismouseDownT)
                }else {
                    if(!this.isAltKeyDown || !this.isShiftKeyDown) {
                        for(let i=0; i< this.slicedClassimagesList.length; i++) {
                        this.slicedClassimagesList[i].isChoosed = false;
                        }
                        this.slicedClassimagesList[index].isChoosed = true;
                        item.index = index;
                        this.choosedSmallImageGroup = [item];
                        // console.log(this.choosedSmallImageGroup)
                        this.setMouseDownPosition(event, item, index);
                    }
                }
                
            },
            changeMouseDownToUp(event) {
                this.ismouseDownT = false;
                this.isHaveCopy = false;
                
                for(let item of this.choosedSmallImageGroup) {
                        let dom = document.getElementById(item.id + "_copy")
                        if(dom) {
                            document.getElementById("dragArea").removeChild(dom)
                            this.isHaveCopy = false;
                        }
                        
                    }
                    if(this.isDraged) {
                        this.changeToTargetClass(event)
                    }


                
                
                // console.log("this.ismouseDownT",this.ismouseDownT)
            },

            // // 鼠标框选开始
            // mousedown(event) {
            //     this.isMouseDown = true;
            //     this.startX = event.clientX;
            //     this.startY = event.clientY;
            //     console.log('SX:',this.startX, 'SY:',this.startY);
            // },
            // mousemove(event) {
            //     let _x = event.clientX;
            //     let _y = event.clientY;
            //     if(this.isMouseDown){
            //         this.updateSelectRect(_x, _y);
            //     }
                
            // },
            // mouseup(event) {
            //     this.isMouseDown = false;
            //     this.endX = event.clientX;
            //     this.endY = event.clientY;
            //     // console.log('EX:',this.endX, 'EY:',this.endY);
            //     document.getElementById("selectdiv").style.display = 'none';
            // },

            // //创建选框
            // createSelectRect(_x, _y) {
            // //    let selectDiv = this.$createElement('div');
            //    let selectDiv = document.createElement('div');
            //    selectDiv.id = "selectdiv";
            //    selectDiv.style.cssText = 'position:absolute;width:0;height:0;margin:0;padding:0;border:1px dashed #eee;background-color:#aaa;z-index:1000;opacity:0.6;display:none;';
            //    document.body.appendChild(selectDiv);
            // //    selectDiv.style.display = 'block';
            // //    selectDiv.style.left = Math.min(_x, this.startX) + 'px';
            // //    selectDiv.style.top = Math.min(_y, this.startY) + 'px';
            // //    selectDiv.style.width = Math.abs(_x - this.startX) + 'px';
            // //    selectDiv.style.height = Math.abs(_y - this.startY) + 'px';

            // },
            //更新框
            // updateSelectRect(_x, _y) {
            // //    let selectDiv = this.$createElement('div');
            // //    let selectDiv = document.getElementById("selectdiv");
            //    let selectDiv = this.$refs.selectDiv;
            //    selectDiv.style.display = 'block';
            //    selectDiv.style.left = Math.min(_x, this.startX) + 'px';
            //    selectDiv.style.top = Math.min(_y, this.startY) + 'px';
            //    selectDiv.style.width = Math.abs(_x - this.startX) + 'px';
            //    selectDiv.style.height = Math.abs(_y - this.startY) + 'px';

            // },




            // 鼠标框选结束
         
            closeCreateNewClassLayer(){
                this.$refs.createNewClassLayer.style.display = "none";
            },
            getAllClasses() {
                
                // for(let i=0; i<this.skusList.length; i++){
                //     console.log("------------");
                //     let flag = 1;
                //     for(let j=0; j<this.classList.length; j++){
                //         if(this.skusList[i].sku_name == this.classList[j].name){
                //             flag = 0;
                //             break;
                //         }
                //     }
                //     if(flag){
                //         this.remainclass.push(this.skusList[i]);
                //     }
                // }
                // this.remainclass = [];
                this.Axios.get(
                    // `${this.api}/index.php/skus/list?type=1`,
                    `${this.api}/${this.getAllClassesUrl}`,
                    {params: {
                            'token':this.token
                        }
                    }
                    
                )
                .then(response =>{

                    if(response.status==200){
                        let _skulist = response.data
                        console.log("_skulist:", _skulist)
                        for(let i=0; i<_skulist.length; i++ ){
                             _skulist[i].name = _skulist[i].sku_name;
                            if(_skulist[i].url){
                                // console.log(_skulist[i].url);
                                _skulist[i].url = `${this.api}` + _skulist[i].url.slice(1)
                               
                            }else {
                                this.skusList[i].url = require('@/assets/img/errorimg.png');
                                
                            }
                        
                        }
                        this.allClasses = _skulist;
                        // console.log('追加后skusList:',this.allClasses);
                    }
                    
                    
                })
                .catch(error => {
                    console.log(error);
                })
                // console.log('remainclass:',this.remainclass);
            },
            completeNewClassName(item){
                this.newClassName = item.sku_name;
            },
            getCreateNewClassLayer(){
                this.getAllClasses();
                this.newFolder = '';
                this.$refs.createNewClassLayer.style.display = "block";
                document.getElementById("newClassInput").focus();
            },
            createnewclass() {
                this.scrollToTopRight()
                this.$refs.createNewClassLayer.style.display = "none";
                let flag = false;
                let tempObject = {};
                let params = {};
                params.name = this.newFolder;
                params.dir = this.poolId;

                for(let i=0; i<this.classList.length; i++) {
                    if(this.newFolder == this.classList[i].name) {
                        flag = true;
                        tempObject = this.classList[i];
                        break;
                    }  
                }
                // console.log(this.newFolder, this.classList, "flag:",flag, "tempObject:",tempObject)
                if(flag) {
                    for(let i = 0; i<this.TargetClassList.length; i++){
                        if(this.TargetClassList[i].name == this.newFolder) {
                            this.TargetClassList.splice(i, 1);
                            break;
                        }
                    }
                    this.TargetClassList.splice(0, 0, tempObject);
                    this.newFolder = '';
                }else {
                    this.Axios.post(
                        // `${this.api}/index.php/annotation/view`,
                        `${this.api}/index.php/classimages/mk`,
                        params
                            
                        
                            // params
                        )
                        .then(response =>{
                            // console.log(response);
                            if(response.data.code == 200) {
                                let tempObject = {};
                                tempObject.name = this.newClassName;
                                tempObject.photos = [];
                                tempObject.state = "未清洗";
                                tempObject.url = this.getLogoByName(this.newClassName)
                                this.classList.splice(0, 0, tempObject);
            

                                this.TargetClassList.splice(0, 0, tempObject);
                                this.newFolder = '';
                            }
                        })
                        .catch(error => {
                            console.log(error);
                        })
                }

                
            },
            
  
           bigOriginalImage(){
               this.sizeNumber = Math.round((this.sizeNumber + 0.1) * 100) / 100;
           },
           smallOriginalImage(){
               this.sizeNumber = Math.round((this.sizeNumber - 0.1) * 100) / 100;
           },
            filtrateClass(){
                // console.log(this.value);
                
                
            },
            //重置本地储存数据
            resetLocalStorge() {
                if(this.isResetCleanState){
                    localStorage.removeItem('classState');
                }

            },
            // 本地储存数据
            setLocalStorge(item) {
                // 储存清洗任务信息
                // let classState = [];
                let flag = 1;
                // console.log('setLocalStorge-item:',item);
                let tempObj = {}
                    tempObj.name = item.name;
                    tempObj.state = item.state;
                    for(let i=0; i<this.classState.length; i++){
                        flag = 0;
                        if(this.classState[i].name == item.name) {
                            
                            // console.log('------------');
                            this.classState.splice(i, 1, tempObj);
                            break;
                        }else{
                            // console.log('+++++++++++++');
                            this.classState.splice(0, 0, tempObj);
                            break;
                        }
                    }
                    if(flag){
                       
                        this.classState.splice(0, 0, tempObj);
                    }
                
                // console.log('classState:',this.classState);
                localStorage.setItem('classState', JSON.stringify(this.classState));
                
            },
            //获取本地储存的数据
            getLocalStorge(name) {
                let _state = "未清洗"
                this.classState =JSON.parse(localStorage.getItem('classState') || '[]') ;
                    for(let j=0; j<this.classState.length; j++){
                        if(name == this.classState[j].name){
                            _state = this.classState[j].state
                            break;
                        }
                    }
                return _state;
            },
            changeState(item, index){
                // console.log(item, index);
                // console.log("this.classList", this.classList);
                this.wqxCount = 0;
                this.yqxCount = 0;
                this.dqrCount = 0;
                for(let i=0;i<this.classList.length; i++){
                    // console.log("-----------------")
                    if(this.classList[i].name == item.name){
                        this.classList.splice(i, 1, item);
                        // break;
                    }
                    if(this.classList[i].state == "未清洗") {
                        this.wqxCount = this.wqxCount + 1;
                    }else if(this.classList[i].state == "已清洗") {
                        this.yqxCount = this.yqxCount + 1;
                    }else if(this.classList[i].state == "待确认") {
                        this.dqrCount = this.dqrCount + 1;
                    }

                }
            //    this.classList.splice(index, 1, item);
                this.setLocalStorge(item);

            },
            showBigImage(item, index, event){
              this.thisIconUrl = item.url;
              
              this.$refs.bigImage.style.display = 'block';
              this.mouseX = event.clientX;
              this.mouseY = event.clientY;
            //   console.log(this.mouseX, this.mouseY)

              this.$refs.bigImage.style.top = (this.mouseY-150) + "px";
              this.$refs.bigImage.style.left = (this.mouseX+20) + "px";
            //   this.$refs.bigImage.style.backgroundColor = "red";

            //   console.log(this.$refs.bigImage.style.cssText)

              //根据宽高调整图片大小自适应
              this.$refs.bigImagePhoto.width = 'auto'
              this.$refs.bigImagePhoto.height = 'auto'
            //   console.log('width1:', this.$refs.bigImagePhoto.width)
            //   console.log('width2:', document.getElementById('bigImagePhoto').width)
            //   console.log('height:', this.$refs.bigImagePhoto.height)
            //   this.$refs.bigImagePhoto.width
              
          },
          hideBigImage(){
              this.$refs.bigImage.style.display = 'none';
              this.thisIconUrl = '';
          },
            getLogoByName(name) {
                for(let i=0; i<this.skusList.length; i++) {
                    if(name == this.skusList[i].sku_name) {
                    // console.log(this.skusList[i].url);
                    // this.skusList[i].url = 'http://ubuntu.zhixiang.co:8889' + this.skusList[i].url.slice(1)
                  
                        return this.skusList[i].url;
                    }
                }
            },
            gernerateIdBySmallImageName(item){
                // console.log("+----------------+")
                let results = item.split('_');
                // console.log(results[0] + results[1] + results[2] + results[3] + results[4])
                return 'smallphoto_' + results[0] + results[1] + results[2] + results[3] + results[4];
            },
            gernerateId(item){
                let results = item.name.split('_');
                // console.log(results[0] + results[1] + results[2] + results[3] + results[4])
                return 'smallphoto_' + results[0] + results[1] + results[2] + results[3] + results[4];
            },
            bigsmallPhotos(){
                // this.spSizeNumber = this.spSizeNumber + 0.1;
                this.spSizeNumber = Math.round((this.spSizeNumber + 0.1) * 100) / 100;
            },
            smallsmallPhotos(){
                // this.spSizeNumber = this.spSizeNumber - 0.1;
                this.spSizeNumber = Math.round((this.spSizeNumber - 0.1) * 100) / 100;
            },
            bigOrSmallSmallPhotos(){
                let spWidth = 100 * this.spSizeNumber;
                let spHeight = 100 * this.spSizeNumber;

                // console.log('width:',spWidth);
                if(this.slicedClassimagesList.length > 0) {
                    for(let i=0; i<this.slicedClassimagesList.length; i++){
                    // console.log(i);
                        document.getElementById(this.slicedClassimagesList[i].id).style.width = parseInt(spWidth)  + 'px';
                        document.getElementById(this.slicedClassimagesList[i].id).style.height = parseInt(spHeight)  + 'px';
                        // document.getElementById(this.gernerateId(this.slicedClassimagesList[i])).style.background = 'white';
                        document.getElementById(this.slicedClassimagesList[i].id).style.display = 'block';
                    }
                    // this.$refs.slicedClassimageItem.width = parseInt(spWidth)  + 'px';
                    // this.$refs.slicedClassimageItem.style.height = parseInt(spHeight) + 'px';
                    // this.$refs.slicedClassimageItem.width = this.$refs.slicedClassimageItem.width * this.spSizeNumber;
                }
                
                
                
            },
            handleTargetAreaScroll(){
                // this.annoTop = document.getElementById("show-rect-layer").getBoundingClientRect().top;
                // this.annoLeft = document.getElementById("show-rect-layer").getBoundingClientRect().left;
                // console.log('annoTop:',this.annoTop,'--annoLeft:',this.annoLeft);
                if(document.getElementById("targetScrollArea")) {
                    this.targetTop = document.getElementById("targetScrollArea").scrollTop - 330;
                    this.targetLeft = document.getElementById("targetScrollArea").scrollLeft;
                }
                
                // console.log('targetTop:',this.targetTop,'-targetLeft:',this.targetLeft);

            },
            controlScroll(results){
                let x1 = parseInt(results[1]);
                let y1 = parseInt(results[2]);
                let x2 = parseInt(results[3]);
                let y2 = parseInt(results[4]);
                let divW = document.getElementById("clean-original-image").offsetWidth;
                let divH = document.getElementById("clean-original-image").offsetHeight;
                // console.log(divW, divH);
                // console.log('top:',(y1 + y2 - divH)/2);
                // console.log('left:',(x1 + x2 - divW)/2);

                document.getElementById("clean-original-image").scrollTop = (y1 * this.sizeNumber + y2 * this.sizeNumber - divH)/2;
                document.getElementById("clean-original-image").scrollLeft = (x1 * this.sizeNumber + x2 * this.sizeNumber - divW)/2 ;
                // document.getElementById("clean-original-image").scrollTop = 373;
                // document.getElementById("clean-original-image").scrollLeft = 709;
                
                // this.$nextTick( () => {  
                //     // this.$refs.showrectlayer.scrollTop = 0 ; 
                //     console.log('-------');
                //     document.getElementById("image-layer").scrollTop = 0;              
                // })
                
            },
            removeTargetClass(index){
                this.TargetClassList.splice(index, 1);
                // for(let i=0; i<TargetClassList.length; i++){
                //     if(TargetClassList[i] == item.name)
                // }
            },
            addToTargetClass(item){
                // console.log(item)
                if(item.name) {
                    let tempObject = {};
                    let flag = false;
                    for(let i = 0; i<this.TargetClassList.length; i++){
                        if(this.TargetClassList[i].name == item.name) {
                            flag = true;
                            tempObject = this.TargetClassList[i];
                            break;
                        }
                    }
                    if(flag) {
                        for(let i = 0; i<this.TargetClassList.length; i++){
                            if(this.TargetClassList[i].name == item.name) {
                                this.TargetClassList.splice(i, 1);
                                break;
                            }
                        }
                        this.TargetClassList.splice(0, 0, tempObject);
                        // this.newClassName = '';
                    }else{
                        tempObject.name = item.name;
                        tempObject.url = item.url;
                        this.TargetClassList.splice(0, 0, tempObject);
                        // console.log(this.TargetClassList);
                    }
                }
                // if(this.TargetClassList.length != 0) {
                //     for(let i = 0; i<this.TargetClassList.length; i++){
                //         if(this.TargetClassList[i].name == item.name) {
                //             break;
                //         }else{
                //             tempObject.name = item.name;
                //             this.TargetClassList.splice(0, 0, tempObject);
                //             console.log(this.TargetClassList);
                //         }
                //     }
                // }else{
                //     tempObject.name = item.name;
                //     this.TargetClassList.splice(0, 0, tempObject);
                //     console.log(this.TargetClassList);
                // }
                
            },
            updateOneClassAndshowSmallImages(item, index){
                

                this.nowChoosedClassIndex = index;
                this.nowItem = item;
                this.nowIndex = index;
                this.nowChoosedSmallImageIndex = 0;
                
                this.updateOneClass(item.name, index, item);
                
                // setTimeout(()=>{
                //     console.log("后执行");
                //     this.showSmallImages(item);
                // },0);
            },
            showSmallImages(item){
                this.currentPage = 1;
                this.totalSmallImagesCount = item.photos.length;
                 this.slicedClassimagesList = [];
                 this.cleanAreaScrollToZeros();
                for(let i=0; i<item.photos.length; i++){
                    let tempObject = {};
                    tempObject.className = item.name;
                    tempObject.url = item.url;
                    tempObject.name = item.photos[i];
                    tempObject.id = this.gernerateIdBySmallImageName(item.photos[i]);
                    tempObject.isChoosed = false;
                    let _namesb = item.name.replace(/%/g, "%25").replace(/\+/g, "%2B")
                
                    tempObject.url = `${this.api}/uploads/classimages/`+ this.poolId + '/' + _namesb + '/' + item.photos[i];
                    // console.log("小图url:", tempObject.id)
                    // this.slicedClassimagesList.splice(0, 0, tempObject);
                    this.totalSmallImages.splice(0, 0, tempObject);
                }
                this.pagingSmallImages();
                // console.log(this.slicedClassimagesList);
                setTimeout(()=>{
                    // this.loadSmallImages()
                    //获取第一张小图的原图
                   
                    this.getOriginalImage(this.slicedClassimagesList[0], 0);

                    this.bigOrSmallSmallPhotos(this.slicedClassimagesList);
                }, 0)
                
                
            },
            drag(event, item, index ) {
                // console.log(event);
                // console.log(item);
                // console.log(item ,'---', index);
                let data = {
                    "item": item,
                    "index": index,
                }
                data = JSON.stringify(data)
                event.dataTransfer.setData("Text", data);
                dom = event.currentTarget
                
            },
            changeClass(smallImageNameList,oldClass, targetClass) {
                // event.preventDefault();
                // let data = event.dataTransfer.getData("Text");
                // // console.log('data:',data);
                // data = JSON.parse(data)
                // // console.log('data:',data);
                
                // // console.log('name:',data.item.name);
                // event.target.appendChild(dom);
                
                // console.log("className:", data.item.className);
                let params ={};
                //处理聚类后的文件
                if(oldClass.indexOf("clustered") != -1) {
                    params.dirpath = oldClass;
                    params.old = "其他";
                }else{
                    params.old = oldClass;
                }
                params.new = targetClass.name;
                params.filename = smallImageNameList;
                params.dir = this.poolId;
                 this.Axios.post(
                        // `${this.api}/index.php/annotation/view`,
                        `${this.api}/index.php/classimages/mv`,
                        params
                        
                    )
                    .then(response =>{
                        // console.log(response);
                        if(response.data.code == 200) {
                            this.$message({
                            message: '更改分类成功！',
                            type: 'success'
                            });
                            // this.updateOneClassAndshowSmallImages(this.nowItem, this.nowIndex);

                            setTimeout(()=>{
                                // document.getElementById(this.gernerateId(data.item)).style.background = 'red';
                                // document.getElementById(this.gernerateId(data.item)).style.display = 'none';
                                // document.getElementById('cleanClassimagesArea').appendChild(document.getElementById(this.gernerateId(data.item)));
                                // document.getElementById('cleanRight').removeChild(document.getElementById(this.gernerateId(data.item)));
                                for(let smallImage of this.choosedSmallImageGroup) {
                                    for(let i=0; i< this.slicedClassimagesList.length; i++) {
                                        
                                            if(this.slicedClassimagesList[i].id == smallImage.id) {
                                            this.slicedClassimagesList.splice(i, 1)
                                        }
                                        }
                                    
                                }
                            },10);
                            
                        }else{
                            this.errorAlert(response.data.msg);
                        }
                    })
                    .catch(error => {
                        console.log(error);
                        this.errorAlert(error.response.data.msg);
                    //     setTimeout(()=>{
                    // // document.getElementById('smallphoto_' + String(data.index)).style.display = 'none';
                    //         document.getElementById('cleanClassimagesArea').appendChild(document.getElementById('smallphoto_' + String(data.index)));
                    //     },1000);
                    })
            },
            updateOneClass(name, index, item){
                let params = {};
                params.name = name;
                params.dir = this.poolId;
                this.Axios.post(
                        // `${this.api}/index.php/annotation/view`,
                        `${this.api}/index.php/classimages/getone-list`,
                        params
                        
                    )
                    .then(response =>{
                        // console.log(response);
                        if(response.data.code == 200) {
                        //    console.log('刷新单个分类成功');
                           let resData = response.data.data;
                            // console.log(resData);
                            for(let key in resData) {
                                let tempObject = {};
                                tempObject.name = key;
                                tempObject.url = this.getLogoByName(key);
                                tempObject.photos = resData[key];
                                tempObject.state = item.state;
                                // console.log(key, ':', resData[key]);
                                
                                // this.classList.push(tempObject)
                                // console.log(key,tempObject.photos);
                                // this.classList.splice(index, 1, tempObject);
                                this.skusfilter.splice(index, 1, tempObject);
                            }
                            setTimeout(()=>{
                                // this.showSmallImages(this.classList[index]);
                                this.showSmallImages(this.skusfilter[index]);
                            },0);
                            
                        }
                    })
                    .catch(error => {
                        console.log(error);
                    })
            },
            errorAlert(tips){
                this.$alert(tips, '更改分类失败', {
                                confirmButtonText: '确定',
                                callback: action => {
                                    this.updateOneClass(this.nowItem.name, this.nowIndex, this.nowItem);
                                    // this.updateOneClass(params.new);

                                    this.$message({
                                    type: 'info',
                                    message: `action: ${ action }`
                                    });
                                }
                            });
            },
            allowDrop(event) {
                event.preventDefault();
            },
            getOriginalImage(item, index){
                // this.isAltKeyDown = false;
                if(this.slicedClassimagesList.length > 0) {
                    for(let i=0; i< this.slicedClassimagesList.length; i++) {
                    this.slicedClassimagesList[i].isChoosed = false;
                    }
                    this.slicedClassimagesList[index].isChoosed = true;
                    item.index = index;
                    this.choosedSmallImageGroup = [item]
                    this.nowChoosedSmallImageIndex = index;
                    this.choosedSlicedImageName = item.name;
                    let results = item.name.split('_');
                    this.nowresults = results
                    let targetImg = parseInt(results[0]);
                    // console.log(this.choosedSmallImageGroup);
                    if(this.nowCleanImgIndex != targetImg){
                        this.getSearchImage(targetImg, results)
                    }else{
                        this.ShowRect(results);
                    }
                }
                
                    

            },
            getSearchImage(imgIndex, results) {
                imgIndex = parseInt(imgIndex);
                this.nowCleanImgIndex = imgIndex;
                this.taskImgUrl = 'http://annotation.lingmou.ai:8000/'+ this.getimgurl +'file=' + imgIndex + '.jpg';
                // console.log(this.taskImgUrl);
                this.Axios.get(
                        // `${this.api}/index.php/annotation/view`,
                        `${this.api}${this.getannourl}`,
                        {params: {
                                // 'token':this.token,
                                'image': imgIndex,
                            }
                        }
                            
                        
                        // params
                    )
                .then(response =>{
                      console.log(response)
                    if(response.status==200){
                        this.bboxes = response.data.bboxes;
                    }
                        
                        

                        // console.log('获得的标注结果：',this.bboxes);
                        
                        
                    this.ShowRect(results);
                        
                })
                .catch(error => {
                        console.log(error)
                       
                    })
            this.createImageLayer(results);
        
            },
            createImageLayer(results) {
                let imgCanvas = this.$refs.imagelager;
                let ctx = imgCanvas.getContext('2d');

                let rectCanvas = this.$refs.showrectlayer;
                let rectCtx = rectCanvas.getContext('2d');

                let img = new Image();
                img.onload = function() {
                    
                    let imgW = img.width * this.sizeNumber;
                    let imgH = img.height * this.sizeNumber;
                    // console.log('图片宽高：',imgW, imgH);

                    imgCanvas.width = imgW  ;
                    imgCanvas.height = imgH  ;

                    ctx.drawImage(img, 0, 0, imgW, imgH);

                    rectCanvas.width = imgW;
                    rectCanvas.height = imgH;
                    
                    this.ShowRect(results);
                }.bind(this)
                img.src = this.taskImgUrl;
                
                
            },
            ShowRect(results){
                // console.log('++++++++');
                let rectCanvas = this.$refs.showrectlayer;
                let rectCtx = rectCanvas.getContext('2d');
                
                //清除canvas上面的框
                rectCtx.clearRect(0,0,rectCanvas.width,rectCanvas.height);

                let rectData = this.bboxes;    
                for(let i=0; i<rectData.length; i++){
                    if(rectData[i].x1 == results[1] && rectData[i].y1 == results[2] && rectData[i].x2 == results[3] && rectData[i].y2 == results[4]){
                        rectCtx.beginPath();
                        rectCtx.strokeStyle = 'rgb(255,0,0)';
                        rectCtx.lineWidth = 2;
                        rectCtx.strokeRect(
                            rectData[i].x1  * this.sizeNumber, rectData[i].y1  * this.sizeNumber,
                            rectData[i].x2 * this.sizeNumber - rectData[i].x1  * this.sizeNumber,
                            rectData[i].y2  * this.sizeNumber - rectData[i].y1  * this.sizeNumber
                        ); 
                        // console.log('框已更新：',rectData[i].x1,rectData[i].y1,rectData[i].x2,rectData[i].y2);                       
                        break;
                    }
                }
                            
                this.controlScroll(results);          
                        

                
            },
            moveChoosedSI(event) {
                let mouseX = event.clientX;
                let mouseY = event.clientY;
                
                // console.log("+++++++++:",this.ismouseDownT)
                if(this.ismouseDownT) {
                    if(!this.isHaveCopy) {
                        this.copyChoosedSI();
                        this.isHaveCopy = true;
                    }
                    this.isDraged = true;
                    // console.log("this.choosedSmallImageGroup:",this.choosedSmallImageGroup)
                    let transX = mouseX - this.mouseDownX;
                    let transY = mouseY - this.mouseDownY;
                    for(let item of this.choosedSmallImageGroup) {
                        // console.log(transX,transY)
                        let dom = document.getElementById(item.id + "_copy")
                        dom.style.zIndex = 2;
                        dom.style.transform = `translate(${transX}px, ${transY}px)`
                    }
                }
            },

        }
        
    }
</script>

<style  scoped>
#addItemToBtn {
   position: fixed;
   top: 82px;
   right: 2px;
}
#targetScrollArea {
    position: absolute;
    top: 330px;
    left: 0px;
    right: 0px;
    bottom: 0px;
    overflow: scroll;
    background-color: whitesmoke;
}
#nowTargetItemDropArea {
    position: absolute;
    top: 80px;
    left: 0px;
    width: 300px;
    height: 250px;
    background: white;
    /* object-fit: contain; */
}
#nowTargetItemDropArea img {
    height: 100%;
    width: 100%;
    object-fit: contain;
}
#tradeFilter {
    position: absolute;
    top: 0px;
    left: 0px;
    width: 300px;
    height: 40px;
}
#searchingPanel {
    z-index: 1;
    position: absolute;
    top: 80px;
    left: 0px;
    right: 0px;
    bottom: 0px;
    background: white;
    overflow: auto;
    display: none;
    border-left: 1px solid #eee;
}
#searchingPanelIterm {
    width: 285px;
    border: 1px solid violet;
}
#searchingPanelIterm p{
    margin-left: 5px;
    font-size: 16px;
    line-height: 22px;
    cursor: pointer;
}
#targetClassSearch {
    position: absolute;
    /* background-color: violet; */
    top: 40px;
    left: 0rem;
    right: 0rem;
    height: 20rem;
}
#targetClassSearchInput {
    position: absolute;
    top: 0rem;
    left: 0rem;
    right: 0px;
}
#targetClassSearchRefresh {
    position: absolute;
    top: 0px;
    left: 194px;
    right: 0px;

}
#targetClassSearchRefreshBtn  {
    width: 106px;
    margin-top: 0px;
    
}
#smallTobigImg {
    position: absolute;
    top: 100px;
    left: 0px;
    right: 0px;
    bottom: 50px;
   
}
#smallTobigImg img {
    width: 100%;
    height: 100%;
    object-fit: contain;
    
}
#wqxClassCount {
    position: absolute;
    top: 30px;
    left: 5px;
}
#wqxClassCount p{
    color: white;
    font-size: 16px;
    line-height: 18px;
    text-align: left;
}
#yqxClassCount {
    position: absolute;
    top: 30px;
    left: 140px;
}
#yqxClassCount p{
    color: white;
    font-size: 16px;
    line-height: 18px;
    text-align: left;
}
#dqrClassCount {
    position: absolute;
    top: 30px;
    left: 260px;
}
#dqrClassCount p{
    color: white;
    font-size: 16px;
    line-height: 18px;
    text-align: left;
}

#smallImagesCount {
    position: absolute;
    top: 5px;
    left: 150px;
}
#smallImagesCount p{
    color: white;
    font-size: 16px;
    line-height: 18px;
    text-align: left;
}
#totalClassCount {
    position: absolute;
    top: 5px;
    left: 5px;
}
#totalClassCount p{
    color: white;
    font-size: 16px;
    line-height: 18px;
    text-align: left;
}
#dragArea {
    position: absolute;
    left: 40rem;
    top: 0rem;
    bottom: 0rem;
    right: 0rem;
    overflow: hidden;
}
#smallImagesGroup {
    background-color: aquamarine;
    width: 300px;
    height: 300px;
    position: absolute;
    top: 10px;
    left: 10px;
}
#cleanArea {
    background-color: rgb(226, 59, 59);
    /* position: absolute;
    top: 0rem;
    left: 0rem; */

}
#showSIBI {
    z-index: 4;
    width: 400px;
    height: 400px;
    /* background-color: rgb(255, 255, 255); */
    /* background-color: black; */
    position: absolute;
    display: none;

}
#showSIBI img {
    width: 100%;
    height: 100%;
    object-fit: contain;
}
#smallImageName {
    font-size: 14px;
    line-height: 15px;
    font-weight: bold;

}
.smallImageChoosed {
    background-color: rgba(0,0,0,0.7);
    box-sizing: border-box;
    border: red 1px solid;
    color: red;
}
.choosedStyle {
    background-color: rgba(255,0,0,0.5);
}
#testButton {
    position: absolute;
    top: 80px;
}
#image-layer {
    position: absolute;
    top: 0px;
    left: 0px;
    z-index: 1;
    margin-left: 0px;
}
#show-rect-layer {
    position: absolute;
    top: 0px;
    left: 0px;
    z-index: 2;
}
#big-small-smallPhotos {
    position: absolute;
    /* top: 150px; */
    width: 100px;
    height: 50px;
    text-align: center;
    bottom: 0px;
    right: 0px;
    background-color: rgba(0,0,0,0.5);
    /* right: 10px; */
    /* left: 10px; */
}
#big-smallPhotos {
    position: absolute;
    font-size: 30px;
    /* width: 30px; */
    margin: 10px;
}
#small-smallPhotos {
    position: absolute;
    font-size: 30px;
    /* width: 30px; */
    margin: 10px;
}

#class-image img {
    width: 100%;
    height: 100%;
    object-fit: contain;

}

#bigImage {
    z-index: 1;
    /* float: right; */
    position: absolute;
    top: 30px;
    left: 30px;
    height: 300px;
    width: 300px;
    display: none;
    background-color: white;
    overflow: hidden;
    text-align: center;
    box-shadow: 10px 10px 5px #888888;
    border: 1px solid #eee;
    padding: 5px;
    
}
#bigImage img {
    /* height: 300px; */
    /* width: 300px; */
    width: 100%; 
    height: 100%; 
    object-fit: contain;
    
}
.class-accept-image img {
    width: 100%;
    height: 100%;
    object-fit: contain;

}
#cleanRight img {
    width: 100%;
    height: 100%;
    object-fit: contain;
}
#originalImageBigSmall{
    /* z-index: 4; */
    width: 100px;
    height: 50px;
    background-color: rgba(0,0,0,0.5);
    position: absolute;
    bottom: 0px;
    left: 0px;
    text-align: center;
    
    /* vertical-align: middle; */
}
#bigOriginalImage {
    position: absolute;
    margin: 10px;
}
#smallOriginalImage {
    position: absolute;
    margin: 10px;
}
#classListClick {
    /* background-color: #888888; */
    width: 130px;
    position: absolute;
    top: 0px;
    right: 0px;
    bottom: 0px;
    cursor: pointer;

}
#createnewclass {
    position: absolute;
    width: 200px;
    bottom: 0px;
    left: 150px;
}
#createNewClassLayer {
    z-index: 3;
    position: absolute;
    left: 50%;
    top: 50%;
    background-color: #888888;
    bottom: 0px;
    width: 600px;
    height: 500px;
    margin-top: -250px;
    margin-left: -300px;
    display: none;
}
#createNewClassLayer #title {
    position: absolute;
    top: 5px;
    left: 50%;
    margin-left: -44px;
    color: white;
    font-size: 22px;
}
#createNewClassLayer .el-icon-close {
    position: absolute;
    font-size: 30px;
    color: white;
    right: 5px;
    top: 5px;
    cursor: pointer;
}
.create-nc-input {
    position: absolute;
    top: 40px;
    left: 100px;
    width: 400px;
}
#newClassNameTips {
    position: absolute;
    top: 80px;
    left: 100px;
    right: 100px;
    height: 350px;
    overflow: auto;
    border-left: #eee 1px solid;
    border-right: #eee 1px solid;
}
#newClassNameTips .item-row {
    height: 30px;
    /* background-color: aqua; */
    border-bottom: #eee 1px solid;
    padding-top: 7px;
    padding-left: 5px;
}
#newClassNameTips .item-row .item-row-name {
    /* position: absolute; */
    font-size: 16px;
    line-height: 16px;
    color: white;
    /* top: 0px; */
}
#newClassBtn {
    position: absolute;
    width: 400px;
    bottom: 10px;
    left: 100px;

}
#selectDiv {
    position:absolute;
    width:0;
    height:0;
    margin:0;
    padding:0;
    border:1px dashed #eee;
    background-color:#aaa;
    z-index:1000;
    opacity:0.6;
    display:none;
}
#pagination {
    position: fixed;
    left: 805px;
    bottom: 0px;
    right: 316px;
    background-color: white;
}
</style>


// WEBPACK FOOTER //
// src/components/pages/CleanClassesTool.vue