---
layout: post
title: 关于iview-admin中table组件render渲染列内容中clipboard的使用
categories: Tech
description: none
keywords: iview-admin, table, clipboard
---

## 问题场景

在iview-admin的table中列内容需要重写，自定义列内容

## template内容编辑

    <Table :columns="columns" :data="this.curChannelDetail.direct_streams"></Table>

引入table组件，columns和data分别为配置项及显示列数据

## javascript内容编辑

以下是所有render的配置属性：

    render: (h, params) {//params是一个对象，包含 row、column 和index，分别指当前行数据，当前列数据，当前行索引
    return h('div', {
    props: { // Component props
      msg: 'hi',
    },
    
    attrs: { // normal HTML attributes
      id: 'foo'
    },
    
    domProps: { // DOM props
      innerHTML: 'bar'
    },
    
    on: { // Event handlers
      click: this.clickHandler
    },
    // For components only. Allows you to listen to native events, rather than events emitted from the component using vm.$emit.
    nativeOn: {
      click: this.nativeClickHandler
    },
    
    class: { // class is a special module, same API as `v-bind:class`
      foo: true,
      bar: false
    },
    
    style: { // style is also same as `v-bind:style`
      color: 'red',
      fontSize: '14px'
    },
    // other special top-level properties
    key: 'key',
    ref: 'ref',
    // assign the `ref` is used on elements/components with v-for
    refInFor: true,
    slot: 'slot'
    })
    }

table组件input值的绑定和获取：
render函数不能使用v-modal，使用以下方法解决

    render: (h, {row}) => {
    return h('Input', {
      props: {
        clearable: true,
        placeholder: '标签编号',
        value: row.tagCode,
      },
      on: {
        input: (val) => {
          row.tagCode = val;
        }
      }
    });
  }

以下为引入 clipboard 并使用的方法

1、npm 安装包

    npm i clipboard -S
  
2、引入包文件

    import Clipboard from 'clipboard'

3、render 中使用的地方绑定值

    render: (h, params) => {
            return h(
              'Input',
              {
                props: {
                  value: params.row.direct_rtmp_play_url
                },
                attrs: {
                  id: 'copy' + params.row.direct_rtmp_id
                }
              },
              [
                h(
                  'Button',
                  {
                    attrs: {
                      // 普通html属性
                      'data-clipboard-target':
                        '#copy' + params.row.direct_rtmp_id
                    },
                    on: {
                      click: () => {
                        this.copy()
                      }
                    },
                    class: {
                      copyBtn: true
                    },
                    ref: 'copy',
                    slot: 'append'
                  },
                  '复制'
                ),
                h(
                  'Button',
                  {
                    props: {
                      // 组件属性
                    },
                    attrs: {
                      // 普通html属性
                      // 'v-clipboard': 'clipOptions'
                    },
                    slot: 'append'
                  },
                  '二维码'
                )
              ]
            )
          }

通过 data-clipboard-target 绑定input组件的id值

4、触发copy方法

    copy() {
      const clipboard = new Clipboard('.copyBtn')
      //成功回调
      clipboard.on('success', e => {
        console.log('ff')
        e.clearSelection()
      })
      this.$Message.success('复制成功')

      //失败回调
      clipboard.on('error', function(e) {})
    }

以上。