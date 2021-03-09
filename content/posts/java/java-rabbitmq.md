---
weight: 4400
title: "Java RabbitMQ"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Java RabbitMQ"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Java, Note]
categories: [Java-MicroServices]

lightgallery: true

toc:
  auto: false
---

## 常用案例

手动确认，如果消费失败，则丢弃消息，不对消息进行重复消费，主要是注释相关的行
（通常我们认为消息一定会正常消费，如果失败就会无限失败的场景适用这种策略）

```java
@Component
@RabbitListener(queues = "eos_monitor_center_sky_walking_queue")
public class SwMqReceiver {

    @Resource(name = "SwAlarmJudge")
    private AlarmJudge alarmJudge;

    @RabbitHandler
    public void receiver(String content, Channel channel, Message message) throws IOException {
        try {
            ObjectMapper mapper = new ObjectMapper();
            MqDataDTO<SwAlarmMessageDTO> mqDataDTO = mapper.readValue(content,
                    new TypeReference<MqDataDTO<SwAlarmMessageDTO>>() {});
            alarmJudge.judge(mqDataDTO);
            // 业务处理成功后调用，消息会被确认消费
            channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
        } catch (Exception ex) {
            // 业务处理失败后调用
            channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, false);
            log.error("【消息被丢弃】add failed: " + ex.getMessage() + content);
        }
    }
}
```