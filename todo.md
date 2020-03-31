1. layout_margin vs layout_marginLeft and other layout_marginXXX
    layout_margin wins
    avoid set layout_margin in styles

2. layout_marginStart, layout_marginEnd > layout_marginLeft, layout_marginRight
  once layout_marginStart/End is set, left and right will be ignored

3. Measure process
  WrapContent (FrameLayout)
    matchparent (TextView)

    custom view override onMeasure

4. Fragment

5. kotlin
  5.1 protected keyword access


  6. virtual tables may not be altered sqlite

  7. kotlin inline class

8. Diacritics
  8.1 UTF-8
  8.2 Diacritics' impact on string length
  8.3 charAt
