meta标签

    <meta name="viewport" content="width=device-width, initial-scale=1.0">

>meta标签的name和content属性可以把设备的viewpoint设置为ideal viewpoint

>如果不设置meta viewport标签，那么移动设备上浏览器默认的宽度值为800px，980px，1024px等这些，总之是大于屏幕宽度的。而每个移动设备浏览器中都有一个理想的>宽度，这个理想的宽度是指css中的宽度，跟设备的物理宽度没有关系，在css中，这个宽度就相当于100%的所代表的那个宽度。我们可以用meta标签把viewport的宽度设为
>那个理想的宽度，如果不知道这个设备的理想宽度是多少，那么用device-width这个特殊值就行了，同时initial-scale=1也有把viewport的宽度设为理想宽度的作用。

输入框和按扭的标签

    <input type="text" placeholder="denis@getcraftwork.com">
    <input class="button" type="button" value="SIGN UP">

    input[type="text"] // 也可以在css中设置属性

@font-face规则

    @font-face{
        font-family: 'Montserrat-ExtraLight';
        src: url('fonts/Montserrat-ExtraLight.otf');
    }

flex属性

    flex-flow: row wrap;

背景图片
    
    background-image: url('images/Background-1.jpg');

组合选择符

    后代选择器(a b)
    子元素选择器(a > b）
    相邻兄弟选择器（a + b）
    普通兄弟选择器（a ~ b）

边框属性

    border-radius: 2px; // 上右下左的圆角尺寸都为2px，也可以分别设置
    box-shadow：10px 10px 5px #888888; // 水平阴影的位置，垂直阴影的位置，阴影的尺寸，阴影的颜色
    border-image：~; // 用图片做边框

背景透明

        background: none;

article/section/div

>在H5中，article元素可以看成是一种特殊类型的section元素，它比section元素更强调独立性。即section元素强调分段或分块，而article强调独立性。具体来说，如果一>块内容相对来说比较独立的、完整的时候，应该使用article元素，但是如果你想将一块内容分成几段的时候，应该使用section元素。另外，在HTML5中，div元素变成了一种>容器，当使用CSS样式的时候，可以对这个容器进行一个总体的CSS样式的套用。

模糊程度

    opacity: 0.7;
    Opacity of an element's text, where 1 is opaque and 0 is entirely transparent.

光标cursor

    cursor: pointer; // 一只手
    cursor:help;
    cursor:wait;

鼠标移动到图片时呈现另一张图片:

```html
<div class="user">
    <div class="user-image"><img src="images/Userpic1.jpg" alt="Userpic1"></div>
    <div class="cover">
        <h6>Evan</h6>
        <p>Senior Developer</p>
        <div>
            <img class="icon" src="images/Facebook.png" alt="Facebook">
            <img class="icon" src="images/Twitter.png" alt="Twitter">
            <img class="icon" src="images/Instagram.png" alt="Instagram">
        </div>
    </div>
</div>
```

```css
#page-six .user {
    position: relative;
}

#page-six .user-image img {
    width: 470px;
    height: 340px;
    margin: 25px 15px;
    cursor: pointer;
}

/* 为cover设置样式 */

#page-six .cover {
    position: absolute;
    top: 25px;  /* 与父元素同步 */
    left: 15px;
    z-index: 1;  
    /* z-index 属性设置元素的堆叠顺序。拥有更高堆叠顺序的元素总是会处于堆叠顺序较低的元素的前面。
    元素可拥有负的 z-index 属性值。Z-index 仅能在定位元素上生效（例如 position:absolute;）！ */
    width: 470px; 
    height: 340px;
    color: #fff;
    background-color: rgba(20, 20, 20, 0.5);
    cursor: pointer;
    opacity: 0;  /* 透明度设为0 */
}
#page-six .cover:hover{
    opacity: 1;  /* hover是改变透明度显示照片 */
}
```

textarea标签

    <textarea name="message" cols="30" rows="10" placeholder="Message"></textarea>
    col表示宽度，rows表示高度

把img元素放入块中后，用flex可以很容易实现图片在左/上，文本在右/下的布局

    <div>
        <div class="icon"><img class="icon" src="images/MapIcon.png" alt="MapIcon"></div>
        <div>360 King Street<br/>Feastrvale Trevose, PA 19057</div>
    </div>

指定父元素的资源数

    <!-- 首先找到p元素的父级元素，再从父级元素从上往下找 -->
    p:first-child
    p:nth-child(n)
    <!-- 从下往上 -->
    p:nth-last-child(n) 
    <!-- 首先找到p元素的父级元素，再从父级元素的该类型元素往下找 -->
    p:first-of-type
    p:nth-of-type(n)
    <!-- 从小往上 -->
    p:nth-last-of-type(n)

display属性

    display: inline-block;
    display: inline-flex;
