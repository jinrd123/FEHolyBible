# 目标

基于`Swiper@5`，并且使用[HTML版本](https://swiperjs.com/swiper-api)（非`Vue / React`版本），完成了如下轮播的定制：

![image-20230423155835858](./记录img/轮播图.png)



# 安装

`pnpm i swiper@5`

组件内：

~~~js
import Swiper from 'swiper';
import "swiper/css/swiper.css"
~~~



# 代码

`html`

~~~html
<!-- 最外层这个home-recommend-carousel不属于Swiper的结构要求，一般包裹Swiper作为我们要放置Swiper的container，起到一个对Swiper模块整体定位以及对模块整体大小控制的作用 -->
<div class="home-recommend-carousel">
    <div class="swiper" ref="recommendSwiper">
        <div class="swiper-wrapper">
          	<!-- swiper-slide是Swiper的轮播item的要求的必须的类，其他的类以及内部的结构就是根据我们自己的需求定制的结构与样式了 -->
            <div 
                class="swiper-slide carousel-border"
                v-for="card in recommedData"
                :key="card.id"
            >
                <div class="carousel-card">
                    <div class="carousel-card-name">{{ card.name }}</div>
                    <div class="carousel-card-department">{{ card.department }}</div>
                    <div class="carousel-card-recommendation">{{ card.recommendation }}</div>
                </div>
            </div>
        </div>
        <!-- 如果需要分页器 -->
        <div class="swiper-pagination"></div>
        
        <!-- 如果需要导航按钮 -->
        <div class="swiper-button-prev"></div>
        <div class="swiper-button-next"></div>
        
    </div>
</div>
~~~

`css`

~~~less
// 如上效果图是一个大模块，里面分为上下结构的title和carousel
.home-recommend {
    width: 100%;
    height: 688px;
    min-width: @div-size; // min-width即为安全距离（稳定呈现的部分）
    background: #fff;
    &-title {
      // ...
    }
    &-carousel {
        margin: 0 auto; // 子模块水平居中，这样就完成了安全距离的效果——静态布局部分一直在中间稳定呈现
        height: 100%;
        width: @div-size; // 宽度等于安全距离宽度，这个盒子作为.swiper的父盒子，宽度也决定了Swiper的宽度
        position: relative;
        overflow: hidden; // 设置超出部分隐藏即可，防止出现过多轮播item，至此，Swiper的整体布局（定位）就完成了，剩下就是对内部样式的定制化了
        .swiper {
          	// .swiper不设置宽度应该就是100%
            position: relative; // 观察Swiper的结构会发现，.swiper-wrapper（轮播item的包裹容器）和.swiper-pagination（分页器，即表示当前item与总item的小点点）和.swiper-button-prev（翻页器）都是.swiper下的同级子结构，所以给.swiper设置相对定位，方便调整它们的位置（定制化）
            height: 557px; // 给整个.swiper一个高度
            .carousel-border {
                // 这里就是我自己卡片的样式了，注意一点即可，宽度严格按照设计稿即可，不要设置margin-left和margin-right，轮播item的间隙在js中激活Swiper时用配置项进行设置
            }
            .swiper-button-prev { // 向前翻页的结构
              	// Swiper默认提供了一个蓝色箭头，发现其实是.swiper-button-prev的::after的伪元素，我们想换成我们自己的图片或者样式，非常容易，直接::after设置为display: none，相当于把它提供的干扰我们样式的dom直接删除
                position: absolute; // 绝对定位，把翻页器移动到我们想要的位置即可，但是这里需要注意我使用了top属性进行定位，而没用bottom，因为swiper的默认样式里是通过top设置的翻页器的位置，并且默认样式在我样式的后面（把我的样式覆盖掉），所以我的bottom不会生效，并且即使使用top进行定位，也需要!important防止被覆盖
                top: 500px !important;
                width: 36px;
                height: 36px;
                background-image: url("../../assets/images/homePage/arrowToLeft.png"); // 使用我们自己的图片，完成样式定制
                background-size: 36px 36px;
                &::after {
                    display: none;
                }
            }
          	// 同上
            .swiper-button-next {
                position: absolute;
                top: 500px !important;
                width: 36px;
                height: 36px;
                background-image: url("../../assets/images/homePage/arrowToRight.png");
                background-size: 36px 36px;
                &::after {
                    display: none;
                }
            }
          	// 分页器，即小圆点
            .swiper-pagination-bullets {
                bottom: 50px; // 定位属性
                .swiper-pagination-bullet {
                    transition: all 0.2s;
                    width: 10px;
                    height: 10px;
                    margin: 0 18px; // 小圆点的空隙用margin设置的，我们这里重新设置一下即可
                }
              	// 激活状态的样式
                .swiper-pagination-bullet-active {
                    width: 12px;
                    height: 12px;
                    background: #3485ff;
                }
            }
        }
				
      	// 中心卡片（被激活的卡片）的样式
        .swiper-slide-active,
        .swiper-slide-duplicate-active {
            transform: scale(1.06); // 变大一点，即异形要求
            &.carousel-border {
							// 卡片本身的一些样式
            }
        }
    }
}
~~~

`js`

~~~js
// 轮播图初始化方法，在mounted中进行调用即可（因为这里的轮播数据非异步数据，所以mountd中dom已经存在，可以进行激活）
initRecommendSwiper() {
    const mySwiper = new Swiper(this.$refs.recommendSwiper, {
    direction: "horizontal", // 轮播item水平排列，默认就是horizontal
    loop: true, // 循环模式选项，即无限循环——最后一个item的下一个是第一个
    slidesPerView: 3, // 预览数为3，即在div.swiper结构的宽度范围中，能看见几个轮播item，如果包裹div.swiper的容器不设置overflow: hidden，那么还会看到更多，所以这个属性某种程度来讲就是形容轮播item大小的属性，他会根据div.swiper的宽度与slidesPerView的值（结合spaceBetween）进行计算每一个轮播item的宽度。（按照设计稿来设置大小设置即可）
    centeredSlides: true, // 开启之后被激活的轮播item处于div.swiper的中心位置
    spaceBetween: 50, // 之所以在样式中不要给轮播item设置水平方向的margin，就是因为这个配置项，轮播item之间的间隔通过这个属性设置，优点像column-clap那个属性，即指定卡片之间的间隔

    // 如果需要分页器
    pagination: {
        el: ".swiper-pagination",
    },

    // 如果需要前进后退按钮
    navigation: {
        nextEl: ".swiper-button-next",
        prevEl: ".swiper-button-prev",
    },

    // 如果需要滚动条
    scrollbar: {
        el: ".swiper-scrollbar",
    },
});
~~~

