---
layout: default
title: 一些在JAVA中要注意的东西
summary: 这两天做LOSP工程，接触JAVA比较多，也遇到了一些问题，所以需要和大家分享一下。<br />首先是关于字符的。那天我在做日历的农历显示，使用字符串的方式来保存节日和节气信息，但出现了很严重的错误，不是日期不对就是节日名称不对……后来折腾半天才终于反应过来，JAVA和C不一样，JAVA中一个汉字算一个字符，而不是两个。<br />接下来，是关于switch...case语句。这个语句蛋疼的一点在于
---
这两天做LOSP工程，接触JAVA比较多，也遇到了一些问题，所以需要和大家分享一下。

首先是关于字符的。那天我在做日历的农历显示，使用字符串的方式来保存节日和节气信息，但出现了很严重的错误，不是日期不对就是节日名称不对……后来折腾半天才终于反应过来，JAVA和C不一样，JAVA中一个汉字算__一个__字符，而不是两个。

接下来，是关于switch...case语句。这个语句蛋疼的一点在于，如果虚拟机在执行的时候，发现一个case符合条件，那么它就会从这个case开始，__一直顺序执行到最后__，无论下面有没有case，也无论后面的case是否符合条件，总之一直执行下去，直到遇到break，或到达整个语句的结束  
例如：  
{% highlight java %}
int i = 2;
switch (i) {
    case 1:
        Log.d("this is 1");
    case 2:
        Log.d("this is 2");
    case 3:
        Log.d("this is 3");
    case 4:
        Log.d("this is 4");
}
{% endhighlight %}

执行后，你会发现，log中输出的是  
> this is 2  
> this is 3  
> this is 4  
是你赋值错了吗？  
不是的，这就是java的特性，case语句中如果没有遇到break就一直执行到末尾，无视其他case  
解法也很简单，只需这样  
{% highlight java %}
int i = 2;
switch (i) {
    case 1:
        Log.d("this is 1");
        break;
    case 2:
        Log.d("this is 2");
        break;
    case 3:
        Log.d("this is 3");
        break;
    case 4:
        Log.d("this is 4");
        break;
}
{% endhighlight %}

再次执行，你会发现，这时候的输出就只剩一个 *this is 2* 了，没有其他的坑爹输出了。

接下来，是另一个容易搞错的东西。在java中，类的成员函数变量可以和类成员变量重名，优先调用本函数的变量。这就导致混淆。因此，调用类成员变量时，如果你不确定有木有重名，我建议还是加个 __this__ 开头比较保险。

另外，在JAVA中执行一些语句，比如文件读写、socket操作时，你会发现莫名其妙的编译错误，其实这是因为，你没有对这些语句的错误进行捕捉然后抛出。在写这些语句的时候，最好这样：  
{% highlight java %}
try {
    ....
} catch (Exception e) {
    ....
}
{% endhighlight %}
这样写，既可以对程序进行更好的错误处理，又可以防止编译错误信息的产生。

在Android的JAVA中，还有一个问题，如果你想在一个软件包中调用其他软件包的JAVA类，那么你就需要在被调用的那个软件包的MK文件中，新增一个静态JAVA库，把需要被外部调用的JAVA文件加进去，然后在调用这个类的源码的MK文件中新增调用这个静态库，然后才能正常调用。

以上均是小弟肤浅的见解，大神飘过即可。

在本文的最后，附送大家一个Lunar.java源码，其中包含了一个Lunar类，是我用在LOSP日历中的一段源码，基于网上的源码改造而来，用于生成农历日期、公历和农历节日，以及二十四节气。其实本文遇到的很多问题都是在写这段源码的时候遇到的。

{% highlight java %}
package com.android.calendar.lunar;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;

import android.content.Context;

public class Lunar {
    private int year;
    private int month;
    private int day;
    private int dow;
    private int m;
    private int d;
    private int ye;
    private boolean leap;
    private Context mContext;
    private String[] chineseNumber;
    private String[] lunarMonthName;
    private SimpleDateFormat chineseDateFormat;
    private static final double D = 0.2422;
    private final static Map<String, Integer[]> INCREASE_OFFSETMAP = new HashMap<String, Integer[]>();
    private final static Map<String, Integer[]> DECREASE_OFFSETMAP = new HashMap<String, Integer[]>();
    final static long[] lunarInfo = new long[] { 0x04bd8, 0x04ae0, 0x0a570,
            0x054d5, 0x0d260, 0x0d950, 0x16554, 0x056a0, 0x09ad0, 0x055d2,
            0x04ae0, 0x0a5b6, 0x0a4d0, 0x0d250, 0x1d255, 0x0b540, 0x0d6a0,
            0x0ada2, 0x095b0, 0x14977, 0x04970, 0x0a4b0, 0x0b4b5, 0x06a50,
            0x06d40, 0x1ab54, 0x02b60, 0x09570, 0x052f2, 0x04970, 0x06566,
            0x0d4a0, 0x0ea50, 0x06e95, 0x05ad0, 0x02b60, 0x186e3, 0x092e0,
            0x1c8d7, 0x0c950, 0x0d4a0, 0x1d8a6, 0x0b550, 0x056a0, 0x1a5b4,
            0x025d0, 0x092d0, 0x0d2b2, 0x0a950, 0x0b557, 0x06ca0, 0x0b550,
            0x15355, 0x04da0, 0x0a5d0, 0x14573, 0x052d0, 0x0a9a8, 0x0e950,
            0x06aa0, 0x0aea6, 0x0ab50, 0x04b60, 0x0aae4, 0x0a570, 0x05260,
            0x0f263, 0x0d950, 0x05b57, 0x056a0, 0x096d0, 0x04dd5, 0x04ad0,
            0x0a4d0, 0x0d4d4, 0x0d250, 0x0d558, 0x0b540, 0x0b5a0, 0x195a6,
            0x095b0, 0x049b0, 0x0a974, 0x0a4b0, 0x0b27a, 0x06a50, 0x06d40,
            0x0af46, 0x0ab60, 0x09570, 0x04af5, 0x04970, 0x064b0, 0x074a3,
            0x0ea50, 0x06b58, 0x055c0, 0x0ab60, 0x096d5, 0x092e0, 0x0c960,
            0x0d954, 0x0d4a0, 0x0da50, 0x07552, 0x056a0, 0x0abb7, 0x025d0,
            0x092d0, 0x0cab5, 0x0a950, 0x0b4a0, 0x0baa4, 0x0ad50, 0x055d9,
            0x04ba0, 0x0a5b0, 0x15176, 0x052b0, 0x0a930, 0x07954, 0x06aa0,
            0x0ad50, 0x05b52, 0x04b60, 0x0a6e6, 0x0a4e0, 0x0d260, 0x0ea65,
            0x0d530, 0x05aa0, 0x076a3, 0x096d0, 0x04bd7, 0x04ad0, 0x0a4d0,
            0x1d0b6, 0x0d250, 0x0d520, 0x0dd45, 0x0b5a0, 0x056d0, 0x055b2,
            0x049b0, 0x0a577, 0x0a4b0, 0x0aa50, 0x1b255, 0x06d20, 0x0ada0 };

    final static String[] Festivals = new String[] { "0101元旦", "0214情人节",  "0308妇女节", 
           "0312植树节", "0401愚人节", "0422地球日", "0501劳动节", "0504青年节", "0531无烟日",
           "0601儿童节", "0606爱眼日", "0701建党日", "0801建军节", "0910教师节", "1001国庆节",
           "1031万圣节", "1111光棍节", "1224平安夜", "1225圣诞节"};

    final static String[] lunarFestivals = new String[] { "腊月三十除夕", "正月初一春节", "正月十五元宵节",
           "五月初五端午节", "七月初七七夕节", "七月十五中元节", "八月十五中秋节", "九月初九重阳节",
           "十月十五下元节", "腊月初八腊八节", "腊月廿四小年"};

    final static String[] SolarTerms = new String[] { "立春", "雨水", "惊蛰", "春分", "清明",
           "谷雨", "立夏", "小满", "芒种", "夏至", "小暑", "大暑", "立秋", "处暑", "白露", 
           "秋分", "寒露", "霜降", "立冬", "小雪", "大雪", "冬至", "小寒", "大寒"};

    static {
        DECREASE_OFFSETMAP.put("雨水", new Integer[] { 2026 });
        INCREASE_OFFSETMAP.put("春分", new Integer[] { 2084 });
        INCREASE_OFFSETMAP.put("小满", new Integer[] { 2008 });
        INCREASE_OFFSETMAP.put("芒种", new Integer[] { 1902 });
        INCREASE_OFFSETMAP.put("夏至", new Integer[] { 1928 });
        INCREASE_OFFSETMAP.put("小暑", new Integer[] { 1925, 2016 });
        INCREASE_OFFSETMAP.put("大暑", new Integer[] { 1922 });
        INCREASE_OFFSETMAP.put("立秋", new Integer[] { 2002 });
        INCREASE_OFFSETMAP.put("白露", new Integer[] { 1927 });
        INCREASE_OFFSETMAP.put("秋分", new Integer[] { 1942 });
        INCREASE_OFFSETMAP.put("霜降", new Integer[] { 2089 });
        INCREASE_OFFSETMAP.put("立冬", new Integer[] { 2089 });
        INCREASE_OFFSETMAP.put("小雪", new Integer[] { 1978 });
        INCREASE_OFFSETMAP.put("大雪", new Integer[] { 1954 });
        DECREASE_OFFSETMAP.put("冬至", new Integer[] { 1918, 2021 });
        INCREASE_OFFSETMAP.put("小寒", new Integer[] { 1982 });
        DECREASE_OFFSETMAP.put("小寒", new Integer[] { 2019 });
        INCREASE_OFFSETMAP.put("大寒", new Integer[] { 2082 });
    }

    private static final double[][] CENTURY_ARRAY = {
            { 4.6295, 19.4599, 6.3826, 21.4155, 5.59, 20.888, 6.318, 21.86, 6.5, 22.2, 7.928, 23.65, 8.35, 23.95, 8.44,
                    23.822, 9.098, 24.218, 8.218, 23.08, 7.9, 22.6, 6.11, 20.84 },
            { 3.87, 18.73, 5.63, 20.646, 4.81, 20.1, 5.52, 21.04, 5.678, 21.37, 7.108, 22.83, 7.5, 23.13, 7.646,
                    23.042, 8.318, 23.438, 7.438, 22.36, 7.18, 21.94, 5.4055, 20.12 } };

     private  int yearDays(int y) {
        int i, sum = 348;
        for (i = 0x8000; i > 0x8; i >>= 1) {
            if ((lunarInfo[y - 1900] & i) != 0)
                sum += 1;
        }
        return (sum + leapDays(y));
    }

     private  int leapDays(int y) {
        if (leapMonth(y) != 0) {
            if ((lunarInfo[y - 1900] & 0x10000) != 0)
                return 30;
            else
                return 29;
        } else
            return 0;
    }

     private  int leapMonth(int y) {
        return (int) (lunarInfo[y - 1900] & 0xf);
    }

     private  int monthDays(int y, int m) {
        if ((lunarInfo[y - 1900] & (0x10000 >> m)) == 0)
            return 29;
        else
            return 30;
    }

     public String animalsYear() {
        final String[] Animals =  new String[]{"鼠", "牛", "虎", "兔", "龙", "蛇", "马", "羊", "猴", "鸡", "狗", "猪"};
        return Animals[(year - 4) % 12];
    }

     private  String cyclicalm(int num) {
        final String[] Gan = new String[]{"甲", "乙", "丙", "丁", "戊", "己", "庚", "辛", "壬", "癸"};
        final String[] Zhi = new String[]{"子", "丑", "寅", "卯", "辰", "巳", "午", "未", "申", "酉", "戌", "亥"};
        return (Gan[num % 10] + Zhi[num % 12]);
    }

     public String cyclical() {
        int num = year - 1900 + 36;
        return (cyclicalm(num));
    }


    public Lunar(Calendar cal,Context context) {
        @SuppressWarnings("unused")
        int yearCyl, monCyl, dayCyl;
        int leapMonth = 0;
        mContext = context;
        m = cal.get(Calendar.MONTH) + 1;
        d = cal.get(Calendar.DAY_OF_MONTH);
        dow = cal.get(Calendar.DAY_OF_WEEK) + 1;
        ye = cal.get(Calendar.YEAR);
        chineseNumber = new String[]{"一", "二", "三", "四", "五", "六", "七", "八", "九", "十", "十一", "十二"};
        lunarMonthName = new String[]{"正", "二", "三", "四", "五", "六", "七", "八", "九", "十", "冬", "腊"};
        String format1 = "yyyy年MM月dd日";
        chineseDateFormat = new SimpleDateFormat(format1);
        Date baseDate = null;
        try {
            String format2 = "1900年1月31日";
            baseDate = chineseDateFormat.parse(format2);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        int offset = (int) ((cal.getTime().getTime() - baseDate.getTime()) / 86400000L);
        dayCyl = offset + 40;
        monCyl = 14;
        int iYear, daysOfYear = 0;
        for (iYear = 1900; iYear < 2050 && offset > 0; iYear++) {
            daysOfYear = yearDays(iYear);
            offset -= daysOfYear;
            monCyl += 12;
        }
        if (offset < 0) {
            offset += daysOfYear;
            iYear--;
            monCyl -= 12;
        }
        year = iYear;
        yearCyl = iYear - 1864;
        leapMonth = leapMonth(iYear);
        leap = false;
        int iMonth, daysOfMonth = 0;
        for (iMonth = 1; iMonth < 13 && offset > 0; iMonth++) {
            if (leapMonth > 0 && iMonth == (leapMonth + 1) && !leap) {
                --iMonth;
                leap = true;
                daysOfMonth = leapDays(year);
            } else
                daysOfMonth = monthDays(year, iMonth);
            offset -= daysOfMonth;
            if (leap && iMonth == (leapMonth + 1))
                leap = false;
            if (!leap)
                monCyl++;
        }
        if (offset == 0 && leapMonth > 0 && iMonth == leapMonth + 1) {
            if (leap) {
                leap = false;
            } else {
                leap = true;
                --iMonth;
                --monCyl;
            }
        }
        if (offset < 0) {
            offset += daysOfMonth;
            --iMonth;
            --monCyl;
        }
        month = iMonth;
        day = offset + 1;
    }

    public  String getChinaDayString(int day) {
        String chineseTen[] = new String[]{"初", "十", "廿", "卅"};
        int n = day % 10 == 0 ? 9 : day % 10 - 1;
        if (day > 30)
            return "";
        else if (day == 10)
            return "初十";
        else if (day == 20)
            return "二十";
        else if (day == 30)
            return "三十";
        else
            return chineseTen[day / 10] + chineseNumber[n];
    }

    public String getFestival() {
        String lunarMonth = lunarMonthName[month - 1] + "月";
        String lunarDay = getChinaDayString(day);
        String lunarDate = lunarMonth + lunarDay;
        String MonthText = null;
        String DayText = null;
        String DateText = null;
        String ret = null;
        Calendar cal = Calendar.getInstance();
        if (m < 10) MonthText = "0" + String.valueOf(m);
        else              MonthText = String.valueOf(m);

        if (d < 10) DayText = "0" + String.valueOf(d);
        else              DayText = String.valueOf(d);

        DateText = MonthText + DayText;
        int i;
        for (i = 0; i < Festivals.length; i++) {
            if (Festivals[i].substring(0, 4).equals(DateText)) {
                ret = Festivals[i].substring(4, Festivals[i].length());
                break;
            }
        }
        if (monthDays(year, 12) == 29) lunarFestivals[0] = "腊月廿九除夕";
        else                           lunarFestivals[0] = "腊月三十除夕";
        if (ret == null) {
            for (i = 0; i < lunarFestivals.length; i++) {
                if (lunarFestivals[i].substring(0, 4).equals(lunarDate)) {
                    ret = lunarFestivals[i].substring(4, lunarFestivals[i].length());
                    break;
                }
            }
        }
        if (ret == null) {
            if (m == 5) {
                cal.set(ye, 4, 1);
                int dow1 = cal.get(Calendar.DAY_OF_WEEK);
                int secondsunday;
                if(dow1 == Calendar.SUNDAY)       secondsunday = 8;
                else secondsunday = 7 - dow1 + Calendar.SUNDAY + 8;
                if (d == secondsunday) ret = "母亲节";
            } else if (m == 6) {
                cal.set(ye, 5, 1);
                int dow1 = cal.get(Calendar.DAY_OF_WEEK);
                int thirdsunday;
                if(dow1 == Calendar.SUNDAY)       thirdsunday = 15;
                else thirdsunday = 7 - dow1 + Calendar.SUNDAY + 15;
                if (d == thirdsunday) ret = "父亲节";
            } else if (m == 11) {
                cal.set(ye, 10, 1);
                int dow1 = cal.get(Calendar.DAY_OF_WEEK);
                int forththursday;
                if(dow1 == Calendar.THURSDAY)       forththursday = 22;
                else forththursday = 7 - dow1 + Calendar.THURSDAY + 22;
                if (d == forththursday) ret = "感恩节";
            }
        }
        if (ret == null) {
            int solarday;
            switch (m) {
                case 1:
                    solarday = getSolarTermNum("小寒");
                    if (solarday == d) ret = "小寒";
                    solarday = getSolarTermNum("大寒");
                    if (solarday == d) ret = "大寒";
                    break;
                case 2:
                    solarday = getSolarTermNum("立春");
                    if (solarday == d) ret = "立春";
                    solarday = getSolarTermNum("雨水");
                    if (solarday == d) ret = "雨水";
                    break;
                case 3:
                    solarday = getSolarTermNum("惊蛰");
                    if (solarday == d) ret = "惊蛰";
                    solarday = getSolarTermNum("春分");
                    if (solarday == d) ret = "春分";
                    break;
                case 4:
                    solarday = getSolarTermNum("清明");
                    if (solarday == d) ret = "清明";
                    solarday = getSolarTermNum("谷雨");
                    if (solarday == d) ret = "谷雨";
                    break;
                case 5:
                    solarday = getSolarTermNum("立夏");
                    if (solarday == d) ret = "立夏";
                    solarday = getSolarTermNum("小满");
                    if (solarday == d) ret = "小满";
                    break;
                case 6:
                    solarday = getSolarTermNum("芒种");
                    if (solarday == d) ret = "芒种";
                    solarday = getSolarTermNum("夏至");
                    if (solarday == d) ret = "夏至";
                    break;
                case 7:
                    solarday = getSolarTermNum("小暑");
                    if (solarday == d) ret = "小暑";
                    solarday = getSolarTermNum("大暑");
                    if (solarday == d) ret = "大暑";
                    break;
                case 8:
                    solarday = getSolarTermNum("立秋");
                    if (solarday == d) ret = "立秋";
                    solarday = getSolarTermNum("处暑");
                    if (solarday == d) ret = "处暑";
                    break;
                case 9:
                    solarday = getSolarTermNum("白露");
                    if (solarday == d) ret = "白露";
                    solarday = getSolarTermNum("秋分");
                    if (solarday == d) ret = "秋分";
                    break;
                case 10:
                    solarday = getSolarTermNum("寒露");
                    if (solarday == d) ret = "寒露";
                    solarday = getSolarTermNum("霜降");
                    if (solarday == d) ret = "霜降";
                    break;
                case 11:
                    solarday = getSolarTermNum("立冬");
                    if (solarday == d) ret = "立冬";
                    solarday = getSolarTermNum("小雪");
                    if (solarday == d) ret = "小雪";
                    break;
                case 12:
                    solarday = getSolarTermNum("大雪");
                    if (solarday == d) ret = "大雪";
                    solarday = getSolarTermNum("冬至");
                    if (solarday == d) ret = "冬至";
                    break;
            }
        }
        return ret;
    }

    public String toString() {
    	String year1 = "年";
    	String run1 = "闰";
    	String month1 = "月";
        return cyclical() + animalsYear() + year1 + (leap ? run1 : "") + lunarMonthName[month - 1] + month1 + getChinaDayString(day);
    }

  public int getSolarTermNum(String name) {


        double centuryValue = 0;

        int centuryIndex = -1;
        if (ye >= 1901 && ye <= 2000) {
            centuryIndex = 0;
        } else if (ye >= 2001 && ye <= 2100) {
            centuryIndex = 1;
        } else {
            throw new RuntimeException("Not supported year：" + ye);
        }
        int i;
        int ordinal = 0;
        for (i = 0; i < SolarTerms.length; i++) {
            if (SolarTerms[i].equals(name)) {
                ordinal = i;
                break;
            }
        }
        centuryValue = CENTURY_ARRAY[centuryIndex][ordinal];
        int dateNum = 0;
        int y = ye % 100;
        if (year % 4 == 0 && year % 100 != 0 || year % 400 == 0) {
            if (name == "小寒" || name == "大寒" || name == "立春" || name == "雨水") {
                y = y - 1;
            }
        }
        dateNum = (int) (y * D + centuryValue) - (int) (y / 4);
        dateNum += specialYearOffset(ye, name);
        return dateNum;
    }

    public int specialYearOffset(int year, String name) {
        int offset = 0;
        offset += getOffset(DECREASE_OFFSETMAP, year, name, -1);
        offset += getOffset(INCREASE_OFFSETMAP, year, name, 1);


        return offset;
    }


    public int getOffset(Map<String, Integer[]> map, int year, String name, int offset) {
        int off = 0;
        Integer[] years = map.get(name);
        if (null != years) {
            for (int i : years) {
                if (i == year) {
                    off = offset;
                    break;
                }
            }
        }
        return off;
    }
}
{% endhighlight %}
