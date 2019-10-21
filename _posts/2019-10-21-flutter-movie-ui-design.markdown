---
layout: post
title:  "Flutter Movie UI Design"
date:   2019-10-21 12:00:00 +0800
categories: flutter
tags: flutter, android, ios
---

  这个Demo使用Flutter的PageView, AnimatedBuilder, Hero, Clipper等主要控件, 详情代码查阅[github][github]

  1. 首页的Banner使用PageView的PageController配合AnimatedBuilder实现滚动大小的变化

      ![demo1](/images/flutter-movie-ui-design-1.jpg)

      AnimatedBuilder配合PageController的代码如下
      ```
      AnimatedBuilder(
        animation: _pageController,
        builder: (BuildContext context, Widget widget) {
          double value = 1;
          if (_pageController.position.haveDimensions) {
            value = _pageController.page - index;
            value = (1 - (value.abs() * 0.3)).clamp(0.0, 1.0);
          }
          return Center(
            child: SizedBox(
              height: Curves.easeInOut.transform(value) * 220.0,
              width: Curves.easeInOut.transform(value) * 400.0,
              child: widget,
            ),
          );
        },
      ```

  2. 使用Hero实现点击Banner的图片动画跳转另一个界面
  3. 详情页面的头部使用Clipper剪切圆形底部，以及使用CustomPatiner实现阴影部分
      ![demo2](/images/flutter-movie-ui-design-2.jpg)

      
      Clipper的实现代码如下
      ```
      class CircularClipper extends CustomClipper<Path> {

        @override
        Path getClip(Size size) {
          var path = Path();
          path.lineTo(0.0, size.height - 50);
          path.quadraticBezierTo(
            size.width / 4,
            size.height,
            size.width / 2,
            size.height,
          );
          path.quadraticBezierTo(
            size.width - size.width / 4,
            size.height,
            size.width,
            size.height - 50,
          );
          path.lineTo(size.width, 0.0);
          path.close();
          return path;
        }

        @override
        bool shouldReclip(CustomClipper<Path> oldClipper) => false;
      }

      @immutable
      class ClipShadowPath extends StatelessWidget {
        final Shadow shadow;
        final CustomClipper<Path> clipper;
        final Widget child;

        ClipShadowPath({
          @required this.shadow,
          @required this.clipper,
          @required this.child,
        });

        @override
        Widget build(BuildContext context) {
          return CustomPaint(
            painter: _ClipShadowShadowPainter(
              clipper: this.clipper,
              shadow: this.shadow,
            ),
            child: ClipPath(child: child, clipper: this.clipper),
          );
        }
      }

      class _ClipShadowShadowPainter extends CustomPainter {
        final Shadow shadow;
        final CustomClipper<Path> clipper;

        _ClipShadowShadowPainter({@required this.shadow, @required this.clipper});

        @override
        void paint(Canvas canvas, Size size) {
          var paint = shadow.toPaint();
          var clipPath = clipper.getClip(size).shift(shadow.offset);
          canvas.drawPath(clipPath, paint);
        }

        @override
        bool shouldRepaint(CustomPainter oldDelegate) {
          return true;
        }
      }
      ```


  [github]: https://github.com/jessyuan24/Flutter-Movie-UI-Design