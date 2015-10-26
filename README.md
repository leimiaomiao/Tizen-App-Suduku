# Tizen-App-Suduku
###概述
Suduku基于Web project开发，本篇主要描述Suduku的核心算法。
###核心算法
Suduku中主要包含7个重要的部分，分别是：
* Algorithm.js
* GameMenu.js
* GameClear.js
* GameLogo.js
* GameRanking.js
* Main.js
* Stage.js

####Main.js
负责游戏的初始化和游戏界面渲染，还负责事件的监听
<pre>
/**
 * Main类
 **/
if(LGlobal.canTouch){
	LGlobal.stageScale = LStageScaleMode.EXACT_FIT;
	LSystem.screen(LStage.FULL_SCREEN);
}

function doScroll() {
	if(window.pageYOffset === 0) {
		window.scrollTo(0, 1);
	}
}
window.onload = function() {
	setTimeout(doScroll, 100);
	init(50,"legend",480,800,main,LEvent.INIT);
}
window.onorientationchange = function() {
	setTimeout(doScroll, 100);
};
window.onresize = function() {
	setTimeout(doScroll, 100);
}


/**层变量*/
//显示进度条所用层
var loadingLayer;
//游戏底层
var backLayer;
var stageLayer;
var labelLayer;
/**数组变量*/
//图片path数组
var imgData = [
{name:"num.0",path:"./images/0.png"},
{name:"num.1",path:"./images/1.png"},
{name:"num.2",path:"./images/2.png"},
{name:"num.3",path:"./images/3.png"},
{name:"num.4",path:"./images/4.png"},
{name:"num.5",path:"./images/5.png"},
{name:"num.6",path:"./images/6.png"},
{name:"num.7",path:"./images/7.png"},
{name:"num.8",path:"./images/8.png"},
{name:"num.9",path:"./images/9.png"},
{name:"return",path:"./images/return.png"},
{name:"close",path:"./images/close.png"},
{name:"logo",path:"./images/logo.png"},
{name:"menu_back",path:"./images/menu_back.png"},
{name:"menu_stage",path:"./images/menu_stage.png"},
{name:"back",path:"./images/back.png"},
{name:"focus",path:"./images/focus.png"},
{name:"window_mini",path:"./images/window_mini.png"},
{name:"num_back",path:"./images/num_back.png"},
{type:"js",path:"./js/Algorithm.js"},
{type:"js",path:"./js/share.js"},
{type:"js",path:"./js/Social.js"},
{type:"js",path:"./js/GameLogo.js"},
{type:"js",path:"./js/GameMenu.js"},
{type:"js",path:"./js/GameClear.js"},
{type:"js",path:"./js/GameRanking.js"},
{type:"js",path:"./js/Stage.js"},
{type:"js",path:"./js/Num.js"}
];
//读取完的图片数组
var imglist = {};
var window_back,window_mini;
var imageArray;
var labelText,nameText,btn_update,rankingLayer;
var stageIndex = 0;
var startTime,stages,times,stagetype;
var btnReturn;
var focus;
var nowNum,selectLayer,stageNumList;
function main(){
	LGlobal.stageScale = LStageScaleMode.SHOW_ALL;
	LSystem.screen(LStage.FULL_SCREEN);
	loadingLayer = new LoadingSample1();
	addChild(loadingLayer);	
	LLoadManage.load(
		imgData,
		function(progress){
			loadingLayer.setProgress(progress);
		},
		gameInit
	);
}
function gameInit(result){
	//LGlobal.setDebug(true);
	imglist = result;
	removeChild(loadingLayer);
	loadingLayer = null;
	//游戏底层添加
	backLayer = new LSprite();
	addChild(backLayer);
	gameLogoShow();
}
function gameStart(index){
	backLayer.removeAllChild();
	backLayer.die();
	stageIndex = index;
	
	stageLayer = new LSprite();
	backLayer.addChild(stageLayer);

	labelLayer = new LSprite();
	backLayer.addChild(labelLayer);
	
	initLabel();
	initStage();
	initButton();
}
function gameLogoShow(){
	var layer = new GameLogo();
	backLayer.addChild(layer);
}

function menuShow(event){
	backLayer.removeAllChild();
	backLayer.die();
	
	var layer = new GameMenu();
	backLayer.addChild(layer);
}
function gameClearShow(){
	stageLayer.mouseChildren = false;
	backLayer.removeEventListener(LEvent.ENTER_FRAME,onframe);
	var layer = new GameClear();
	backLayer.addChild(layer);
}

function initStage(){
	startTime = new Date().getTime();
	stageNumList = new Array();
	var numlist = getNumList();
	var bitmapData = new LBitmapData(imglist["back"]);
	var bitmap = new LBitmap(bitmapData);
	stageLayer.addChild(bitmap);
	var stage = stageMenu[stageIndex];
	for(var i=0;i<9;i++){
		stageNumList.push([]);
		for(var j=0;j<9;j++){
			var num = new Num(numlist[i][j],numlist[i][j] == 0 ? selectShow : null);
			num.x = j*40 + 80;
			num.y = i*40 + 100;
			if(stage.flag == 1){
				num.x += (j/3>>>0)*5 - 5;
				num.y += (i/3>>>0)*5 - 5;
			}
			stageLayer.addChild(num);
			stageNumList[i].push(num);
		}
	}
	
	backLayer.addEventListener(LEvent.ENTER_FRAME,onframe);
}
function onframe(){
	var str = (new Date().getTime() - startTime) + "";
	times.text = str.substr(0,str.length - 3) + "." + str.substr(str.length - 3,1);
}

function selectShow(event,numObject){
	stageLayer.mouseChildren = false;
	nowNum = numObject;
	selectLayer = new LSprite();
	backLayer.addChild(selectLayer);
	var window_back = new LSprite();
	window_back.graphics.drawRect(1,"#000",[0,0,LGlobal.width,LGlobal.height],true,"#000");
	window_back.alpha = 0.7;
	selectLayer.addChild(window_back);
	//window.open(LGlobal.canvasObj.toDataURL());
	window_mini = new LSprite();
	var bitmapData = new LBitmapData(imglist["window_mini"]);
	var bitmap = new LBitmap(bitmapData);
	window_mini.addChild(bitmap);

	window_mini.x = (LGlobal.width - window_mini.getWidth())*0.5;
	window_mini.y = (LGlobal.height + 320 - window_mini.getHeight())*0.5;
	selectLayer.addChild(window_mini);
	window_mini.scaleX = 0.1;
	window_mini.scaleY = 0.1;
	LTweenLite.to(window_mini,0.3,
			{ 
				scaleX:1,
				scaleY:1,
				onUpdate:function(){
					window_mini.x = (LGlobal.width - window_mini.getWidth()*window_mini.scaleX)*0.5;
					window_mini.y = (LGlobal.height + 320 - window_mini.getHeight()*window_mini.scaleY)*0.5;
				},
				onComplete:showWindow,
				ease:Sine.easeIn
			}
		);
}
function showWindow(){
	window_mini.scaleX = 1;
	window_mini.scaleY = 1;
	window_mini.x = (LGlobal.width - window_mini.getWidth())*0.5;
	window_mini.y = (LGlobal.height + 320 - window_mini.getHeight())*0.5;
	for(var i=0;i<3;i++){
		for(var j=0;j<3;j++){
			var num = new Num(i*3+j + 1,changeValue);
			num.x = j*40 + 40;
			num.y = i*40 + 40;
			window_mini.addChild(num);
		}
	}
	var closeButton = new LSprite();
	var closeBitmap = new LBitmap(new LBitmapData(imglist["close"]));
	closeButton.addChild(closeBitmap);
	closeButton.x = closeButton.y = -20;
	window_mini.addChild(closeButton);
	closeButton.addEventListener(LMouseEvent.MOUSE_UP,function(event){
		stageLayer.mouseChildren = true;
		backLayer.removeChild(selectLayer);
	});
}
function changeValue(event,obj){
	nowNum.changeValue(obj.value);
	backLayer.removeChild(selectLayer);
	stageLayer.mouseChildren = true;
	var win = checkWin();
	if(win)gameClearShow();
}

function initLabel(){
	var stage = stageMenu[stageIndex];
	
	stages = new LTextField();
	stages.font = "黑体";
	stages.size = 40;
	stages.x = 253;
	stages.y = 20;
	stages.text = stageIndex+1;
	labelLayer.addChild(stages);

	labelText = new LTextField();
	labelText.font = "黑体";
	labelText.size = 30;
	labelText.x = 100;
	labelText.y = 475;
	labelText.text = "Time:";
	labelLayer.addChild(labelText);
	times = new LTextField();
	times.font = "黑体";
	times.size = 30;
	times.x = 200;
	times.y = 475;
	times.text = "0";
	labelLayer.addChild(times);
	
	stagetype = new LTextField();
	stagetype.font = "黑体";
	stagetype.size = 30;
	stagetype.x = 100;
	stagetype.y = 520;
	stagetype.text = stage.flag == 0?"级别：普通":"级别：高级";
	labelLayer.addChild(stagetype);
	stagetype = new LTextField();
	stagetype.font = "黑体";
	stagetype.size = 20;
	stagetype.x = 90;
	stagetype.y = 570;
	labelLayer.addChild(stagetype);
	if(stage.flag == 0){
		stagetype.text = "说明：横竖没有重复数字";
	}else{
		stagetype.text = "说明：横竖没有重复数字。";
		stagetype = new LTextField();
		stagetype.font = "黑体";
		stagetype.size = 20;
		stagetype.x = 80;
		stagetype.y = 610;
		labelLayer.addChild(stagetype);
		stagetype.text = "每九个方格内也没有重复数字";
	}
	
}
function initButton(){
	var bitmap_up = new LBitmap(new LBitmapData(imglist["return"]));
	var bitmap_down = new LBitmap(new LBitmapData(imglist["return"]));
	bitmap_down.alpha = 0.5;
	btnReturn = new LButton(bitmap_up,bitmap_down);
	backLayer.addChild(btnReturn);
	btnReturn.x = 420; 
	btnReturn.y = 30;
	btnReturn.scaleX = 1;
	btnReturn.scaleY = 1;
	btnReturn.addEventListener(LMouseEvent.MOUSE_UP,menuShow);
}

window.addEventListener('tizenhwkey', function(e) {
	if(e.keyName == "back") {
		try {
			tizen.application.getCurrentApplication().exit();
		} catch (error) {
			console.error("getCurrentApplication(): " + error.message);
		}
	}
},false);
</pre>
####Algorithm.js
负责整个游戏的算法部分，负责生成整个游戏，和计算游戏是否成功。
<pre>
function getNumList(){
	var stage = stageMenu[stageIndex];
	if(stage.flag == 0){
		return randomNum01(stage.lv);
	}else{
		return randomNum02(stage.lv);
	}
}
function randomNum01(lv){
	var i,j,list = new Array(),result = new Array();
	for(i=0;i < 9;i++){
		list.push([1,2,3,4,5,6,7,8,9]);
		for(j=0;j<i;j++)list[i].push(list[i].shift());
		result.push([]);
	}
	list = list.sort(function(a,b){return Math.random()>.5?-1:1;});
	var rand = new Array(0,1,2,3,4,5,6,7,8).sort(function(a,b){return Math.random()>.5?-1:1;});
	for(i=0;i < 9;i++){
		for(j=0;j < 9;j++){
			result[i].push(list[i][rand[j]]);
		}
	}

	for(i=0;i < 9;i++){
		for(j=0;j < lv + 1;j++){
			var ran1 = Math.random()*9 >>> 0;
			result[i][ran1] = 0;
			ran1 = Math.random()*9 >>> 0;
			result[ran1][i] = 0;
		}
	}
	return result;
}
function randomNum02(lv){
	var i,j,k,list = [],result = [],rand;
	for(i=0;i < 9;i++){
		list.push([1,2,3,4,5,6,7,8,9]);
		for(j=0;j < i;j++)list[i].push(list[i].shift());
	}
	list = [list[0],list[3],list[6],list[1],list[4],list[7],list[2],list[5],list[8]];
	
	rand = new Array(0,1,2).sort(function(a,b){return Math.random()>.5?-1:1;}).concat(
			new Array(3,4,5).sort(function(a,b){return Math.random()>.5?-1:1;}),
			new Array(6,7,8).sort(function(a,b){return Math.random()>.5?-1:1;})
		);
	for(i=0;i < 9;i++)result.push(list[rand[i]]);
	list = result;
	rand = new Array(0,1,2).sort(function(a,b){return Math.random()>.5?-1:1;}).concat(
			new Array(3,4,5).sort(function(a,b){return Math.random()>.5?-1:1;}),
			new Array(6,7,8).sort(function(a,b){return Math.random()>.5?-1:1;})
		);
	result = [];
	for(i=0;i < 9;i++){
		result.push([]);
		for(j=0;j < 9;j++){
			result[i].push(list[i][rand[j]]);
		}
	}

	for(i=0;i < 9;i++){
		for(j=0;j < lv + 1;j++){
			var ran1 = Math.random()*9 >>> 0;
			result[i][ran1] = 0;
			ran1 = Math.random()*9 >>> 0;
			result[ran1][i] = 0;
		}
	}
	return result;
}
function checkWin(){
	var check01,check02;
	for(var i=0;i < 9;i++){
		check01 = [];
		check02 = [];
		for(var j=0;j < 9;j++){
			if(stageNumList[i][j].value > 0)check01.push(stageNumList[i][j].value);
			if(stageNumList[j][i].value > 0)check02.push(stageNumList[j][i].value);
		}
		check01 = deleteEleReg(check01);
		check02 = deleteEleReg(check02);
		if(check01.length < 9)return false;
		if(check02.length < 9)return false;
	}
	var stage = stageMenu[stageIndex];
	if(stage.flag){
		return checkWin02();
	}
	return true;
}
function checkWin02(){
	for(var i=0;i < 3;i++){
		for(var j=0;j < 3;j++){
			if(!check_mini(i,j))return false;
		}
	}
	return true;
}
function check_mini(i2,j2){
	var check_arr = [];
	for(var i=i2*3;i < i2*3+3;i++){
		for(var j=j2*3;j < j2*3+3;j++){
			if(check_arr[stageNumList[i][j].value])return false;
			check_arr[stageNumList[i][j].value] = 1;
		}
	}
	return true;
}
function deleteEleReg(s){     
	var a = {};   
  	var len = s.length;   
  	for(var i=0; i < len; i++)  {     
    	if(typeof a[s[i]] == "undefined")     
    	a[s[i]] = 1;     
  	}     
  	s.length = 0;     
  	for(var i in a)     
  	s[s.length] = i;     
  	return s;     
} 
</pre>
