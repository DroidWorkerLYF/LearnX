#Layout
##View的layout
##ViewGroup的onLayout

#draw

1. 绘制背景 background.draw(canvas)
2. 绘制自己onDraw
3. 绘制children dispatchDraw
4. 绘制装饰 onDrawScrollBars

#自定义View

1. 支持wrap_content
2. 注意padding和margin
3. view中有动画或线程需要及时停止,View#onDetachedFromWindow