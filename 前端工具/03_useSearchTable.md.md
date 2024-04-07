---
tags:
  - 工作
  - work
  - 前端
date: 2024年4月3日14:51:06
---

#### useSearchTable

##### 定义以及传参

```tsx
/** 表格页 */  
export const useSearchTable = (config: SearchTableConfig, slots?: Record<string, any>) => {  
  const tableKey = config?.key ?? uuid()  
  /** 刷新数据 */  
  const refresh = () => useSearchTableRefresh(tableKey)  
  /** 静默刷新数据，没有loading */  
  const silentRefresh = () => useSearchTableRefresh(tableKey, true)  
  
  const Table = (  
    <div class="base-table-page-wrapper">  
      <SearchTable tableKey={tableKey} {...config.props} {...config}>  
        {{ ...slots }}  
      </SearchTable>  
    </div>  
  )  return {  
    tableKey,  
    refresh,  
    silentRefresh,  
    Table  
  }  
}
```

注意这里的返回:
```tsx
tableKey,  
refresh,  
silentRefresh,  
Table
```
在使用其中某些返回值时,请使用解构
```tsx
const { Table, refresh, silentRefresh } = useSearchTable({config...})
```
后续在tsx中的调用:
```tsx
export default defineComponent({  
  name: 'StoreInfo',  
  setup() {
      return () => {  
      return (  
        <div class="test-index"> 
          {Table}  
        </div>      
    )
    }  
  }  
})
```


`SearchTableConfig`:
```tsx
export interface SearchTableConfig {  
  /** 表格的key，全局唯一 */  
  key?: string  
  /** 表格标题 */  
  title?: any  
  /** 工具栏 */  
  toolbar?: any  
  filterFront?: any  
  filterBehind?: any  
  /** 过滤器定义 */  
  filter?: FilterDefine  
  dataSourceFormat?: (dataSource: any[]) => any[]  
  /** 表格配置参数，继承自 Antd Table 组件 */  
  table?: OmitedTableProps  
  customTable?: (dataSource: any[]) => any  
  /** 请求地址（或customRequest，至少存在一个） */  
  requestURL?: string  
  /** 导出地址 */  
  exportURL?: string  
  // 默认降序字段，默认 create_time  
  defaultDescs?: string  
  /** 自定义请求（或requestURL，至少存在一个） */  
  customRequest?: (params: RequestParams) => Promise<unknown>  
  /** 拦截请求 */  
  onInterceptRequest?: (config: AxiosRequestConfig<unknown>) => AxiosRequestConfig<unknown>  
  /** 拦截响应 */  
  onInterceptResponse?: (response: AxiosResponse<unknown>) => AxiosResponse<unknown>  
  props?: Record<string, any>  
}
```
##### 关于他的静默刷新

在`useSearchTable`中,`Table`是由`SearchTable`封装的
`SearchTable`:
```jsx
const SearchTable = defineComponent({  
  name: 'SearchTable',  
  props: {  
    tableKey: { type: String },  
    toolbar: {},  
    title: {},  
    filter: {  
      type: Object as PropType<FilterDefine>  
    },  
    filterFront: {},  
    filterBehind: {},  
    requestURL: {  
      type: String  
    },  
    exportURL: {  
      type: String  
    },  
    customRequest: {  
      type: Function  
    },  
    defaultDescs: {  
      type: String  
    },  
    table: {  
      type: Object as PropType<OmitedTableProps>,  
      default: () => ({})  
    },  
    customTable: {},  
    dataSourceFormat: {  
      type: Function  
    }  
  },  
  slots: Object as SlotsType<{  
    default: any  
    title: any  
    toolbar: any  
    filterFront: any  
    filterBehind: any  
  }>,  
  emits: {  
    interceptRequest: (config: AxiosRequestConfig<unknown>) => config,  
    interceptResponse: (response: AxiosResponse<unknown>) => response  
  },  
  setup(props, { attrs, slots }) {  
    const isLoading = ref(false)  
    const dataSourceRef = ref([])  
    const dataSource = computed(() => {  
      try {  
        return props.dataSourceFormat?.(dataSourceRef.value) ?? dataSourceRef.value  
      } catch {  
        return dataSourceRef.value  
      }  
    })  
    const pagination = reactive({  
      size: 20,  
      current: 1,  
      total: 0  
    })  
    const tableRef = ref()  
    const successRef = ref(false)  
    const errorRef = ref(false)  
    const sorterRef = ref<{  
      descs?: string  
      ascs?: string  
    }>({  
      descs: props.defaultDescs ?? 'create_time',  
      ascs: undefined  
    })  
    const fetchData = (params = {}) => {  
      console.log('fetchData触发了!')  
      if (!props.requestURL && !props.customRequest) {  
        message.warning('请设置 requestURL 或 customRequest')  
        return  
      }  
      if (!silent.value) {  
        isLoading.value = true  
      }  
      try {  
        const onResolveHandler = (res: any) => {  
          successRef.value = true  
          errorRef.value = false  
          if (res.message) {  
            message.success(res.message)  
          }  
          isLoading.value = false  
  
          if (Array.isArray(res.data)) {  
            dataSourceRef.value = res.data  
          } else {  
            try {  
              dataSourceRef.value = res.data.records  
              pagination.total = res.data.total  
              pagination.size = res.data.size  
              pagination.current = res.data.current  
            } catch (err) {  
              dataSourceRef.value = []  
              pagination.total = 0  
            }  
          }  
  
          // 数据更新后表格自动滚动到顶部  
          try {  
            if (tableRef.value?.$el) {  
              tableRef.value.$el.querySelector(`.${PREFIX_CLS}-table-body`).scrollTop = 0  
            }  
          } catch (err) {  
            console.log(err)  
          }  
        }  
        const onErrorHandler = (err: any) => {  
          errorRef.value = true  
          successRef.value = false  
          dataSourceRef.value = []  
          useRequestErrorMessage(err)  
          isLoading.value = false  
        }  
  
        const requestParams = {  
          current: pagination.current,  
          size: pagination.size,  
          // ascs: undefined,  
          // descs: undefined,  
          ...sorterRef.value,  
          ...params  
        }  
        if (props.customRequest) {  
          props.customRequest(requestParams).then(onResolveHandler).catch(onErrorHandler)  
        } else {  
          request<{ message: string }, { message: string }>({  
            url: props.requestURL,  
            method: 'get',  
            params: requestParams  
          })  
            .then(onResolveHandler)  
            .catch(onErrorHandler)  
        }  
      } catch (err) {  
        isLoading.value = false  
      }  
    }  
  
    const onSearch = (payload: Record<string, any>) => {  
      pagination.current = 1  
      fetchData({ ...payload })  
    }  
  
    emitter.off(`SearchTableRefresh:${props.tableKey}`)  
    emitter.off(`SearchTableSilentRefresh:${props.tableKey}`)  
    emitter.on(`SearchTableRefresh:${props.tableKey}`, () => {  
      fetchData()  
    })  
  
    const silent = ref(false)  
    emitter.on(`SearchTableSilentRefresh:${props.tableKey}`, () => {  
      silent.value = true  
      fetchData()  
      silent.value = false  
    })  
  
    emitter.off(`SearchTableReload:${props.tableKey}`)  
    emitter.on(`SearchTableReload:${props.tableKey}`, () => {  
      pagination.current = 1  
      fetchData()  
    })  
  
    // 卸载之前取消监听  
    onBeforeUnmount(() => {  
      emitter.off(`SearchTableRefresh:${props.tableKey}`)  
      emitter.off(`SearchTableSilentRefresh:${props.tableKey}`)  
      emitter.off(`SearchTableReload:${props.tableKey}`)  
    })  
  
    const onPaginationChange = (page: number, size: number) => {  
      pagination.current = page  
      pagination.size = size  
      fetchData()  
    }  
  
    onMounted(() => {  
      fetchData()  
    })  
  
    const handleSort = (sorter: ITableSort) => {  
      sorterRef.value = { descs: undefined, ascs: undefined }  
      if (sorter.order) {  
        sorterRef.value[sorter.order === 'desc' ? 'descs' : 'ascs'] = snakeCase(sorter.key)  
      }  
      fetchData()  
    }  
  
    const handleExport = () => {  
      const msgId = uuid()  
      message.loading({ key: msgId, content: '请求创建导出任务中...' })  
      request({  
        url: props.exportURL ?? `${props.requestURL}/export`,  
        method: 'get'  
      })  
        .then((res: any) => {  
          if (res.code === 200) {  
            message.success({ key: msgId, content: res.msg || '已创建导出任务' })  
            emitter.emit('refreshTaskCount')  
          } else {  
            message.error({ key: msgId, content: '该表格不支持导出' })  
          }  
        })  
        .catch((err) => {  
          message.error({ key: msgId, content: '该表格不支持导出' })  
        })  
    }  
    const isFullScreen = ref(false)  
  
    return () => {  
      return (  
        <div class={['base-table-page-container', isFullScreen.value && 'fullscreen']}>  
          <div class="base-table-page">  
            <div class="btp__header">  
              <div class="btp__toolbar">  
                <h2 key="title">{slots.title?.() ?? props.title}</h2>  
                <div class="btp__toolbar-content">{slots.toolbar?.() ?? props.toolbar}</div>  
                <div class="btp__toolbar-split" />  
                <div class="btp__settings">  
                  <div class="btp__setting-btn clickable" onClick={handleExport}>  
                    <Icon name="download" />  
                    <span>导出</span>  
                  </div>  
                  <div                    class={['btp__setting-btn clickable', isFullScreen.value && 'active']}  
                    onClick={() => (isFullScreen.value = !isFullScreen.value)}  
                  >                    <Icon name="border-radius-all"></Icon>  
                    <span>{isFullScreen.value ? '退出全屏' : '全屏'}</span>  
                  </div>  
                  <div class="btp__setting-btn clickable" onClick={() => message.info('设置表格，暂未支持')}>  
                    <Icon name="settings"></Icon>  
                    <span>设置</span>  
                  </div>  
                </div>  
              </div>  
              {props.filterFront ?? slots.filterFront?.()}  
              <Filter filter={props.filter} onSearch={onSearch} />  
              {props.filterBehind ?? slots.filterBehind?.()}  
              <div class="btp__footer">  
                {isLoading.value ? (  
                  <strong class="btp__summary">正在查找中，请稍候...</strong>  
                ) : successRef.value ? (  
                  <strong class="btp__summary">  
                    <Icon class="color-success" name="ok-bold" style="margin-right:4px;"></Icon>  
                    <span>共查找到 {pagination.total} 条数据</span>  
                    {dataSource.value.length > 1 ? (  
                      <span>  
                        ，当前第 {(pagination.current - 1) * pagination.size + 1} ~{' '}  
                        {(pagination.current - 1) * pagination.size + dataSource.value.length} 条  
                      </span>  
                    ) : (  
                      pagination.total > 0 && `当前第 ${(pagination.current - 1) * pagination.size + 1} 条`  
                    )}  
                  </strong>  
                ) : errorRef.value ? (  
                  <div class="btp__error">  
                    <Icon name="error-bold"></Icon>  
                    <strong>抱歉，出现了错误，请稍后重试。</strong>  
                  </div>  
                ) : (  
                  ''  
                )}  
  
                <Pagination  
                  showLessItems  
                  pageSize={pagination.size}  
                  current={pagination.current}  
                  total={pagination.total}  
                  showQuickJumper  
                  showSizeChanger                  onChange={onPaginationChange}  
                />  
              </div>  
            </div>  
            {(props.customTable as any)?.(dataSource.value) ?? (  
              <Table                ref={tableRef}  
                nativeProps={{  
                  ...props.table,  
  
                  dataSource: dataSource.value,  
                  loading: {  
                    size: 'large',  
                    indicator: <Spin />,  
                    spinning: isLoading.value  
                  }  
                }}  
                onSort={handleSort}  
              >  
                {{ ...slots, title: null }}  
              </Table>  
            )}  
          </div>  
        </div>  
      )  
    }  
  }  
})
```
在上面的代码中,`fetchData`是负责通过传入的`requestURL`/`customRequest`去请求数据
通过多个`emitter.on`的监听来触发,除此之外,在`onMounted`阶段有一次`fetchData`的触发:
```tsx
onMounted(() => {  
  fetchData()  
})
```

##### 关于表单的构建
在上面`SearchTable`的代码中,props的table是传入表单的,你需要传入名为 `columns` 的array,例如:

```tsx
table: {  
  columns: [  
    {  
      title: '商品信息',  
      width: 250,  
      fixed: 'left',  
      customRender: ({ record }) => {  
        return (  
          <div class="good-info">  
            <div  
              class="image"  
              style={{ backgroundImage: `url(${record.coverImages?.[0]})` }}  
              onClick={() => {  
                usePreviewImage({ url: record.coverImages?.[0] })  
              }}  
            ></div>  
            <div class="info">  
              <div  
                class="name max-2-rows"  
                onDblclick={() => {  
                  updateGoodName(record.id, record.title)  
                }}  
              >  
                {record.title}  
              </div>  
            </div>  
          </div>  
        )  
      }  
    },  
    {  
      title: '售价（元）',  
      width: 250,  
      customRender: ({ record }) => {  
        return (  
          <div class="GOODS__price">  
            {uniq(compact([record.priceMin, record.priceMax])).join('～')} &emsp;  
            <a onClick={onUpdatePrice}>价格管理</a>  
          </div>  
        )  
      }  
    },  
    {  
      dataIndex: 'type',  
      title: '商品类型',  
      width: 120,  
      align: 'center',  
      customRender: ({ text }) => {  
        return (  
          <div class="good-col__count">  
            {text}|{GOODS_TYPE_OPTIONS.find((item) => item.value === text)?.label}  
          </div>        )  
      }  
    },  
    {  
      dataIndex: 'canSaleStock',  
      title: '可售库存',  
      width: 100,  
      align: 'right'  
    },
   ]
}
```

##### 渲染列中每一个 customRender 的渲染参数

像是下面代码中:
```tsx
{  
  dataIndex: 'canSaleStock',  
  title: '可售库存',  
  width: 100,  
  align: 'right',  
  customRender: (obj) => {  
    console.log(obj)  
    return <div class="good-col__count">{obj.text}</div>  
  }},
```
`customRender`传入的是一个obj,这个obj是该表中这一行的数据,像是:
```json
{
    "text": 0,
    "value": 0,
    "record": {
        "id": "1774743874399420417",
        "title": "暂不上架",
        "type": 0,
        "code": "001",
        "coverImages": [
            {
                "url": "https://dev-cdn.cardcat.cn/kacat/XkvmlJA0CNdB.png",
                "width": 720,
                "height": 1000
            }
        ],
        "sort": 0,
        "status": 0,
        "priceMin": "1.00",
        "priceMax": "1.00",
        "canSaleStock": 0,
        "onPayingStock": 0,
        "soldStock": 0
    },
    "index": 1,
    "renderIndex": 1,
    "column": {
        "dataIndex": "canSaleStock",
        "title": "可售库存",
        "width": 100,
        "align": "right",
        "sorter": false,
        "sortDirections": [
            "descend",
            "ascend"
        ]
    }
}
```
这个对象
> 注意
如果你需要展示的列的值,依赖于其他字段的值,那么可能考虑在`customRender`的匿名函数中直接传入obj,如果不依赖其他值,还是使用解构赋值来获取,像是:
```tsx
{  
  dataIndex: 'status',  
  title: '状态',  
  width: 80,  
  align: 'center',  
  fixed: 'right',  
  customRender: ({ record }) => {  
    return <Switch checked={record.status === COMMON_STATUS_ON} onChange={() => onToggleStatus(record)} />  
  }},
```

##### table中方法的定义
你可以注意到,在上面的表单中有很多function,他们都是在setup中注册的:
```tsx
export default defineComponent({  
  name: 'StoreInfo',  
  setup() {  
    /** 创建商品 */  
    const handleCreateGood = () => {  
      console.log('创建商品')  
    }
    return()
})
```
###### 关于表头的自定义

![[img/1712134924416.png]]
在上面的图中`Table`是通过`SeaarchTable`返回的,那么在`SearchTable`中就可以自定义表格的头部元素了:
![[img/Pasted image 20240403171424.png]]
这里是在setup的return中返回`table`:
```tsx
  return (  
    <div class={['base-table-page-container', isFullScreen.value && 'fullscreen']}>  
      <div class="base-table-page">  
        <div class="btp__header">  
          <div class="btp__toolbar">  
            <h2 key="title">{slots.title?.() ?? props.title}</h2>  
            <div class="btp__toolbar-content">{slots.toolbar?.() ?? props.toolbar}</div>  
            <div class="btp__toolbar-split" />  
            <div class="btp__settings">  
              <div class="btp__setting-btn clickable" onClick={handleExport}>  
                <Icon name="download" />  
                <span>导出</span>  
              </div>  
              <div                class={['btp__setting-btn clickable', isFullScreen.value && 'active']}  
                onClick={() => (isFullScreen.value = !isFullScreen.value)}  
              >                <Icon name="border-radius-all"></Icon>  
                <span>{isFullScreen.value ? '退出全屏' : '全屏'}</span>  
              </div>  
              <div class="btp__setting-btn clickable" onClick={() => message.info('设置表格，暂未支持')}>  
                <Icon name="settings"></Icon>  
                <span>settings!!!</span>  
              </div>  
            </div>  
          </div>  
          {props.filterFront ?? slots.filterFront?.()}  
          <Filter filter={props.filter} onSearch={onSearch} />  
          {props.filterBehind ?? slots.filterBehind?.()}  
          <div class="btp__footer">  
            {isLoading.value ? (  
              <strong class="btp__summary">正在查找中，请稍候...</strong>  
            ) : successRef.value ? (  
              <strong class="btp__summary">  
                <Icon class="color-success" name="ok-bold" style="margin-right:4px;"></Icon>  
                <span>共查找到 {pagination.total} 条数据</span>  
                {dataSource.value.length > 1 ? (  
                  <span>  
                    ，当前第 {(pagination.current - 1) * pagination.size + 1} ~{' '}  
                    {(pagination.current - 1) * pagination.size + dataSource.value.length} 条  
                  </span>  
                ) : (  
                  pagination.total > 0 && `当前第 ${(pagination.current - 1) * pagination.size + 1} 条`  
                )}  
              </strong>  
            ) : errorRef.value ? (  
              <div class="btp__error">  
                <Icon name="error-bold"></Icon>  
                <strong>抱歉，出现了错误，请稍后重试。</strong>  
              </div>  
            ) : (  
              ''  
            )}  
  
            <Pagination  
              showLessItems  
              pageSize={pagination.size}  
              current={pagination.current}  
              total={pagination.total}  
              showQuickJumper  
              showSizeChanger              onChange={onPaginationChange}  
            />  
          </div>  
        </div>  
        {(props.customTable as any)?.(dataSource.value) ?? (  
          <Table            ref={tableRef}  
            nativeProps={{  
              ...props.table,  
  
              dataSource: dataSource.value,  
              loading: {  
                size: 'large',  
                indicator: <Spin />,  
                spinning: isLoading.value  
              }  
            }}  
            onSort={handleSort}  
          >  
            {{ ...slots, title: null }}  
          </Table>  
        )}  
      </div>  
    </div>  
  )
```
像是上面的代码中,表头部分在`Table`这个组件上面,那么表头的搜索条件在:
```tsx
<Filter filter={props.filter} onSearch={onSearch} />
```
###### table的传参
可以参考: 
[ant-design-vue](https://www.antdv.com/components/table-cn)
项目中使用的table是基于这个封装的,传参基本一致










