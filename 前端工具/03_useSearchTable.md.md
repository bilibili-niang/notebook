
#### useSearchTable

##### 定义

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