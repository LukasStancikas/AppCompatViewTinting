# AppCompatViewTinting
Setting a dynamic background to a View, either a Ripple if possible, or lighter color of set background color





fun View.setRippleBackground(
    backgroundColor: Int,
    radius: Float,
    isBorderless: Boolean = false,
    borderColor: Int = Color.TRANSPARENT,
    borderWidth: Int = 1.px.toInt(),
    radii: FloatArray? = null
) {
    val drawable = GradientDrawable()
    drawable.cornerRadius = radius
    radii?.let {
        drawable.cornerRadii = radii
    }
    if (borderColor != Color.TRANSPARENT) {
        drawable.setStroke(borderWidth, borderColor)
    }
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
        drawable.colors = intArrayOf(backgroundColor, backgroundColor)

        val typedValue = TypedValue()
        val rippleColor = if (context.theme.resolveAttribute(
                R.attr.colorControlHighlight,
                typedValue,
                true
            )
        )
            typedValue.data
        else
            Color.TRANSPARENT

        val color = ColorStateList(
            arrayOf(intArrayOf()),
            intArrayOf(rippleColor)
        )
        if (isBorderless) {
            background = RippleDrawable(color, null, drawable)
        } else {
            background = RippleDrawable(color, drawable, drawable)
        }
    } else {
        drawable.setColor(backgroundColor)
        val pressedColor = Color.argb(
            Color.alpha(backgroundColor),
            (Color.red(backgroundColor) * 1.4f).roundToInt(),
            (Color.green(backgroundColor) * 1.4f).roundToInt(),
            (Color.blue(backgroundColor) * 1.4f).roundToInt()
        )
        val colorsStateList = getPressedStateList(backgroundColor,pressedColor)
        val wrapper = DrawableCompat.wrap(drawable)
        DrawableCompat.setTintList(wrapper,colorsStateList)

        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN) {
            background = wrapper
        } else {
            setBackgroundDrawable(wrapper)
        }
    }
}

fun getPressedStateList(backgroundColor: Int, pressedColor: Int): ColorStateList {
    //PRESSED MUST BE BEFORE ENABLED
    val states = arrayOf(
        intArrayOf(android.R.attr.state_pressed),  // pressed
        intArrayOf(android.R.attr.state_enabled) // enabled
    )

    val colors = intArrayOf(pressedColor, backgroundColor)

    return ColorStateList(states, colors)
}


val Int.dp: Int
    get() = (this / Resources.getSystem().displayMetrics.density).toInt()
val Int.px: Float
    get() = (this * Resources.getSystem().displayMetrics.density)
