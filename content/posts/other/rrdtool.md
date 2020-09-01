---
weight: 1
title: "Java review"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Java review"
resources:
- name: "base-image"
  src: "base-image.jpg"

tags: [Java, Note]
categories: [Java]

lightgallery: true

toc:
  auto: false
---

review

<!-- more -->

```java
package com.liuzhidream.rrdtool.service.rrdtool;

import lombok.SneakyThrows;
import org.rrd4j.ConsolFun;
import org.rrd4j.DsType;
import org.rrd4j.core.*;

import java.io.IOException;
import java.util.*;

/**
 * @Description
 * @Author VanLiuZhi
 * @Date 2020-02-14 17:06
 */
public class RrdToolUtil {

    public void created() throws IOException {
        long start = System.currentTimeMillis() / 1000;
        RrdDef rrdDef = new RrdDef("./test2.rrd", start, 1);

        rrdDef.addDatasource("users", DsType.GAUGE, 60 * 2, 0, Double.NaN);
        rrdDef.addDatasource("devices", DsType.GAUGE, 60 * 2, 0, Double.NaN);

        /** 每分钟一个存档, 两年共计 2*365*24*60 */
        rrdDef.addArchive(ConsolFun.AVERAGE, 0.5, 1, 2 * 365 * 24 * 60);

        /** 每小时一个存档, 两年共计2*365*24 */
        rrdDef.addArchive(ConsolFun.AVERAGE, 0.5, 60, 2 * 365 * 24);

        /** 每天一个存档, 两年共计2*365 */
        rrdDef.addArchive(ConsolFun.AVERAGE, 0.5, 60 * 24, 2 * 365);
        RrdDb rrdDb = RrdDb.of(rrdDef);
        rrdDb.close();
    }

    public void update() throws IOException {
        final RrdDb rrdDb = RrdDb.of("./test2.rrd");
        final Random random = new Random();

        Timer timer = new Timer();
        timer.schedule(new TimerTask() {
            @SneakyThrows
            @Override
            public void run() {
                Sample sample = rrdDb.createSample();
                long time = System.currentTimeMillis() / 1000;
                sample.setTime(time);
                sample.setValue("users", random.nextInt(1000));
                sample.setValue("devices", random.nextInt(1000));
                sample.update();
                System.out.println(sample);
            }
        }, 1000, 1000);
    }

    public void fetch() throws IOException {
        final RrdDb rrdDb = RrdDb.of("./test2.rrd");
        FetchRequest request = rrdDb.createFetchRequest(
                ConsolFun.AVERAGE,
                Util.getTimestamp(2020, Calendar.FEBRUARY, 14, 23, 31),
                Util.getTimestamp(2020, Calendar.FEBRUARY, 14, 23, 32), 60);
        FetchData fetchData = request.fetchData();
        double[] values = fetchData.getValues("users");
        System.out.println(Arrays.toString(values));
    }

    public static void main(String[] args) throws IOException {
        RrdToolUtil rrdToolUtil = new RrdToolUtil();
        rrdToolUtil.update();
//        rrdToolUtil.fetch();
    }
}

```