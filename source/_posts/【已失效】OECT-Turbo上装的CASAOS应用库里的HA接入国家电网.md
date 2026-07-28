---
layout: post
title: 【已失效】OECT-Turbo上装的CASAOS应用库里的HA接入国家电网
comments: true
tags:
  - Home Assistant
categories: 
  - 教程
date: 2025-08-31 23:00:31
---
国家电网插件用的是[sgcc_electricity_new](https://github.com/ARC-MX/sgcc_electricity_new)，流程稍微有点麻烦。目前已失效，失效原因是国网登录验证从滑块变成了其他。
<!--more-->
装好CASAOS后，安装软件库里的Home Assistant，安装完常用插件后之后，参考上述教程接入国家电网。
### 准备
1. 在github上下载sgcc_electricity_new的zip程序
2. 通过MobaXterm登录OECT-Turbo后台
3. 将解压后的sgcc_electricity_new-master文件夹放到存放HA插件的路径下，可根据文件夹内是否存在已有的插件名来判断
4. 复制文件夹中的example.env文件，起名.env
5. 修改.evn文件，主要是里面的登录账户、登录密码、homeassistant的长期令牌

### 运行
1. cd到设备sgcc_electricity_new-master文件夹下
2. 键入docker-compose up -d，第一次会提示安装docker-compose，根据提示安装即可（注意一天最多登录5次）
3. 再次cd到设备sgcc_electricity_new-master文件夹下
4. 重复的键入docker-compose logs sgcc_electricity_app，日志刷新可能会比较慢，直到出现User xxxx state-refresh task run successfully!，或者出现报错，就等第二天继续
5. 记录上述出现的xxxx

### 添加集成
1. 打开HA的config文件夹
2. 编辑configuration.yaml文件，在现有内容后添加，xxxx修改为上述记录的xxxx
```
template:
  - trigger:
      - platform: event
        event_type: state_changed
        event_data:
          entity_id: sensor.electricity_charge_balance_xxxx
    sensor:
      - name: electricity_charge_balance_xxxx
        unique_id: electricity_charge_balance_xxxx
        state: "{{ states('sensor.electricity_charge_balance_xxxx') }}"
        state_class: total
        unit_of_measurement: "CNY"
        device_class: monetary

  - trigger:
      - platform: event
        event_type: state_changed
        event_data:
          entity_id: sensor.last_electricity_usage_xxxx
    sensor:
      - name: last_electricity_usage_xxxx
        unique_id: last_electricity_usage_xxxx
        state: "{{ states('sensor.last_electricity_usage_xxxx') }}"
        state_class: measurement
        unit_of_measurement: "kWh"
        device_class: energy

  - trigger:
      - platform: event
        event_type: state_changed
        event_data:
          entity_id: sensor.month_electricity_usage_xxxx
    sensor:
      - name: month_electricity_usage_xxxx
        unique_id: month_electricity_usage_xxxx
        state: "{{ states('sensor.month_electricity_usage_xxxx') }}"
        state_class: measurement
        unit_of_measurement: "kWh"
        device_class: energy

  - trigger:
      - platform: event
        event_type: state_changed
        event_data:
          entity_id: sensor.month_electricity_charge_xxxx
    sensor:
      - name: month_electricity_charge_xxxx
        unique_id: month_electricity_charge_xxxx
        state: "{{ states('sensor.month_electricity_charge_xxxx') }}"
        state_class: measurement
        unit_of_measurement: "CNY"
        device_class: monetary

  - trigger:
      - platform: event
        event_type: state_changed
        event_data:
          entity_id: sensor.yearly_electricity_usage_xxxx
    sensor:
      - name: yearly_electricity_usage_xxxx
        unique_id: yearly_electricity_usage_xxxx
        state: "{{ states('sensor.yearly_electricity_usage_xxxx') }}"
        state_class: total_increasing
        unit_of_measurement: "kWh"
        device_class: energy

  - trigger:
      - platform: event
        event_type: state_changed
        event_data:
          entity_id: sensor.yearly_electricity_charge_xxxx
    sensor:
      - name: yearly_electricity_charge_xxxx
        unique_id: yearly_electricity_charge_xxxx
        state: "{{ states('sensor.yearly_electricity_charge_xxxx') }}"
        state_class: total_increasing
        unit_of_measurement: "CNY"
        device_class: monetary
```
3. 重启HA

### 展示
1. 在概述中随便新建一个实体
2. 打开这个实体的代码编辑器
3. 替换其中的entity，注意替换xxxx
      每日用电量的entity: sensor.last_electricity_usage_xxxx
      电费余额的entity: sensor.electricity_charge_balance_xxxx
      上月电费的entity: sensor.month_electricity_charge_xxxx
      上月用电量的entity: sensor.month_electricity_usage_xxxx
      今年总用电量的entity: sensor.yearly_electricity_usage_xxxx
      今年总用电费用的entity: sensor.yearly_electricity_charge_xxxx
4. 后续可根据需求，复制一个已有的实体，再通过修改其中entity的方式进行界面部署和美化。