# ViewPager2 实现轮播效果

* n+2 实现无限轮播

```kotlin
class BannerAdapter : RecyclerView.Adapter<ViewHolder>() {

    override fun getItemCount(): Int {
        return realCount + 2
    }

    private fun getRealPosition(position: Int) = getRealPosition(position, realCount)

    private fun getRealPosition(position: Int, realCount: Int) = when (position) {
        0 -> realCount - 1
        realCount + 1 -> 0
        else -> position - 1
    }
}
```

```kotlin
class BannerOnPageChangeCallback(private val viewPager2: ViewPager2) : OnPageChangeCallback() {

    private var tempPosition = -1
    private var isScrolled = false

    override fun onPageSelected(position: Int) {
        if (isScrolled) {
            tempPosition = position
        }
    }

    override fun onPageScrollStateChanged(state: Int) {
        when (state) {
            ViewPager2.SCROLL_STATE_DRAGGING, ViewPager2.SCROLL_STATE_SETTLING -> isScrolled = true
            ViewPager2.SCROLL_STATE_IDLE -> {
                isScrolled = false
                when (tempPosition) {
                    0 -> viewPager2.setCurrentItem(realCount, false)
                    realCount + 1 -> viewPager2.setCurrentItem(1, false)
                }
            }
        }
    }
}
```

```kotlin
viewPager2.registerOnPageChangeCallback(BannerOnPageChangeCallback(viewPager2))
viewPager2.setCurrentItem(1, false)
```

* 实现自动轮播

```kotlin
Observable.interval(3, TimeUnit.SECONDS)
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe {
        viewPager2.currentItem = (viewPager2.currentItem + 1) % (realCount + 2)
    }
```

* 实现圆点指示器

```xml
<com.google.android.material.tabs.TabLayout
    android:id="@+id/indicator"
    android:layout_width="match_parent"
    android:layout_height="8dp"
    android:layout_marginBottom="8dp"
    android:background="@android:color/transparent"
    app:layout_constraintBottom_toBottomOf="@id/viewPager2"
    app:layout_constraintEnd_toEndOf="@id/viewPager2"
    app:layout_constraintStart_toStartOf="@id/viewPager2"
    app:tabBackground="@drawable/viewpager_indicators"
    app:tabGravity="center"
    app:tabIndicatorHeight="0dp"
    app:tabMaxWidth="16dp"
    app:tabRippleColor="@android:color/transparent"/>
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
  <item android:drawable="@drawable/viewpager_selected" android:state_selected="true" />
  <item android:drawable="@drawable/viewpager_not_selected" />
</selector>
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">
  <item
      android:height="8dp"
      android:width="8dp">
    <shape android:shape="oval">
      <size
          android:height="8dp"
          android:width="8dp" />
      <solid android:color="#CCCCCC" />
    </shape>
  </item>
</layer-list>
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">
  <item
      android:height="8dp"
      android:width="8dp">
    <shape android:shape="oval">
      <size
          android:height="8dp"
          android:width="8dp" />
      <solid android:color="#FFFFFF" />
    </shape>
  </item>
</layer-list>
```

```kotlin
TabLayoutMediator(indicator, viewPager2) { tab, position ->
    if (position == 0 || position == realCount + 1) {
        tab.view.visibility = View.GONE
    }
}.attach()
```
