ansible-playbook-td-agent
========================

This is playbook sample of td-agent.

## 想定されるサーバ構成
* WEBサーバ（任意の台数）
* 集計サーバ（1台）

## 目的とする動作
WEBサーバ（Nginx）のアクセスログを fluentd で集計サーバに流し、集計サーバ上で Elasticsearch に流し込む。  
得られた集計データを Kibana で閲覧する。

### 設定例
WEBサーバから nginx.access.ltsv タグで送出し、集計サーバでプレフィックス"es"を付与した後Elasticsearchに流す設定です。
* WEBサーバ( group_vars/web )
```yaml:group_vars/web
td_aggrigator_name: log01
td_aggrigator_host: log01.local
td_conf_include:
  - nginx
```
* 集計サーバ( group_vars/log )
```yaml:group_vars/log
td_conf_include:
  - rewrite_tag:
    - elasticsearch
td_es_out_rules:
  - name: access_log
    match: es.nginx.access.ltsv
    logstash_format: True
td_rewrite_tag_rules:
  - match: nginx.**.ltsv
    backref: yes
    rules:
      - key: path
        regex: '.*'
        tag: 'es.${tag}'
```
