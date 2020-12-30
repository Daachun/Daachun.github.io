---
title: 狗屁不通点子生成器
date: 2020-03-19 01:10:19
type: music
comments:
description:
top_img:
mathjax:
katex:
---

Seblague大佬的随机点子生成器

---
<style>
    .prompt {
	font-family: opensans;
	color: black;
	text-align: center;
	/*font-size:15px;*/

    position: relative;
    height: 150px;
    margin：0 auto;
	/*outline: 2px solid #555555;*/
}

.button {
	background-color: #4CAFFF;
	
	font-family: opensans;
	text-align: center;
	font-size: 25px;
	padding: 10px 0px;
	color: white;

	outline-style:none;
	border:none;
	cursor:pointer;

	position:relative;
	width:50%;
	bottom:10%;
    left:25%;
    
	-webkit-touch-callout: none; /* iOS Safari */
	-webkit-user-select: none; /* Safari */
	-moz-user-select: none; /* Old versions of Firefox */
	-ms-user-select: none; /* Internet Explorer/Edge */
	user-select: none; /* Chrome, Opera and Firefox */
	touch-action: manipulation; 
}

.button:hover {
  background-color: #5dc2f5;
}
.button:active {
  background-color: #5591ff;
}
</style>
<script src="/IdeaGenerator/data.js"></script>
<script src="/IdeaGenerator/generator.js"></script>


<div class="prompt" id="content">...</div>

<button class="button" onclick="generate()">生成点子</button>





