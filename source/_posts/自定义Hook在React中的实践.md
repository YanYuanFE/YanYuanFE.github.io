---
title: 自定义Hook在React中的实践
date: 2020-08-03 21:59:14
tags:
  - React
  - hooks
---

> Hook 是 React 16.8 的新增特性。它可以让你在不编写 Class 的情况下使用 state 以及其他的 React 特性。

![image](http://img.yanyuanfe.cn/hooks.png)

<!--more-->

通过自定义 Hook，可以将组件逻辑提取到可重用的函数中。

### Hook介绍
React 16.8 新增 Hook特性。它可以让你在不编写 Class 的情况下使用 state 以及其他的 React 特性。

通过自定义 Hook，可以将组件逻辑提取到可重用的函数中。

### 实践自定义Hook

曾经使用React的Class组件的时候，我遇到这样一个问题：在一个前端分页的表格展示页面中，分页数据没有同步到URL上，当刷新页面或者进入详情页再后退的时候，前一个分页的数据丢失，会导致展示为第一页的数据，用户体验很不好。

在hook还未出现的时候，这个问题解决起来是很繁琐的，思路大概是监听Table的onChange事件，获取分页参数和筛选参数，然后调用history.replace同步到URL，解决方法其实很简单，但是无法复用，每个组件中都会充斥大量与业务无关的代码。

直到Hooks的出现。当时看到umi开源了umi-hooks,于是提交了一个issue,看会不会有官方的自定义hook实现:

https://github.com/alibaba/hooks/issues/232

过了一段时间，随着对hook了解的深入，突然有了使用hook来实现的思路，大致代码如下：

``` js
// useQuery.js
import { useEffect, useState } from "react";
import { useHistory } from "react-router-dom";
import qs from "qs";

export const useQuery = (initQuery = {}) => {
  const history = useHistory();
  const url = history.location.search;
  const params = qs.parse(url.split("?")[1]);
  const [query, setQuery] = useState({
    offset: 0,
    ...initQuery,
    ...params
  });
  
  useEffect(() => {
    history.replace(`?${qs.stringify(query)}`);
  }, [query]);

  return [query, setQuery];
};
```
使用：

``` js
import React, { useEffect, useState } from "react";
import { Layout, Button, Table, Input } from "antd";
import "./index.less";
import { doctorService } from "../../services/doctor.service";
import { useQuery } from "../../common/useQuery";

const { Search } = Input;

const DoctorList = () => {
  const [doctorList, setData] = useState([]);
  const [loading, setLoading] = useState(false);
  const [query, setQuery] = useQuery({
    search: ""
  });

  const columns = [
    {
      title: "编号",
      dataIndex: "doctor_id",
      key: "id",
      defaultSortOrder: "descend",
      sorter: (a, b) => a.doctor_id - b.doctor_id
    },
    {
      title: "姓名",
      dataIndex: "name",
      key: "name"
    },
    {
      title: "医院",
      dataIndex: "hospital",
      key: "hospital"
    },
    {
      title: "地区",
      dataIndex: "zone",
      key: "zone"
    },
    {
      title: "销售",
      dataIndex: "sales",
      key: "sales"
    },
    {
      title: "操作",
      dataIndex: "action",
      key: "action",
      render: (_, record) => {
        return (
          <Button href={`#/doctor/${record.guid}${record.doctor_id}/patient`} type="primary">
            查看患者
          </Button>
        );
      }
    }
  ];

  useEffect(() => {
    setLoading(true);
    doctorService.getDoctorList({ company_id: 1 }).then(res => {
      setData(res ? res.doctor : []);
      setLoading(false);
    });
  }, []);

  return (
    <Layout>
      <div className="search">
        <Search
          defaultValue={query.search}
          placeholder="输入名字或者医院搜索"
          onSearch={val => {
            setQuery({
              ...query,
              search: val
            });
          }}
          style={{ width: 200 }}
        />
        <Button href="#/doctor/add" type="primary">
          添加医生
        </Button>
      </div>
      <Table
        loading={loading}
        dataSource={
          query.search
            ? doctorList.filter(
                item => new RegExp(query.search).test(item.name) || new RegExp(query.search).test(item.hospital)
              )
            : doctorList
        }
        columns={columns}
        rowKey="doctor_id"
        pagination={{ defaultPageSize: 20, current: Number(query.offset) }}
        onChange={(pagination, filters, sorter) => {
          setQuery({ ...query, offset: pagination.current });
        }}
      />
    </Layout>
  );
};

export default DoctorList;

```
从上面可以看到，useQuery的代码还是很简单的，useQuery接收一个对象作为初始状态，并使用useState进行内部状态的保存，useQuery的返回值中setQuery可以对query状态进行修改，query的变化通过useEffect进行监听，每次有query变化都使用history.replace同步到URL上，这样当页面进行切换的时候，下一个页面使用history.goBack()回退的时候，上一个页面重新渲染，useQuery重新初始化，初始化的时候从URL上获取query参数，对query state进行初始化，外部的组件也能获取到新的query，从而可以进行搜索或者根据分页开始相应的渲染。

### 高阶组件的实现

使用Hook实现后，我发现在Class组件中，使用高阶组件也能实现这种功能，下面是一个简单的实现：

```
import { React } from "react";
import qs from "qs";

export const withQuery = (initQuery = {}) = (Comp) => {
  constructor(props) {
    const url = props.history.location.search;
    const params = qs.parse(url.split("?")[1]);
    this.state = {
      offset: 0,
      ...initQuery,
      ...params
    }
  }
  changeQuery = (query) => {
    this.setState({
      ...query,
    }, () => {
      this.history.replace(`?${qs.stringify(query)}`);
    })
  }
  render() {
      const {query} = this.state;
      return <Comp {...this.props} query={query} setQuery={this.changeQuery} />
  }
}

```
下面是withQuery的使用，省略部分代码：

```
class List extends React.Component {
  render() {
    const {query, setQuery} = this.props;
    return (
      <Layout>
      <div className="search">
        <Search
          defaultValue={query.search}
          placeholder="输入名字或者医院搜索"
          onSearch={val => {
            setQuery({
              ...query,
              search: val
            });
          }}
          style={{ width: 200 }}
        />
        <Button href="#/doctor/add" type="primary">
          添加医生
        </Button>
      </div>
      <Table
        loading={loading}
        dataSource={
          query.search
            ? doctorList.filter(
                item => new RegExp(query.search).test(item.name) || new RegExp(query.search).test(item.hospital)
              )
            : doctorList
        }
        columns={columns}
        rowKey="doctor_id"
        pagination={{ defaultPageSize: 20, current: Number(query.offset) }}
        onChange={(pagination, filters, sorter) => {
          setQuery({ ...query, offset: pagination.current });
        }}
      />
    </Layout>
    )
  }
}

export default withQuery({search: ""})(List);
```
对比自定义Hook和高阶组件的实现：

- 自定义Hook和高阶组件都可以实现逻辑复用
- 两者都可以在内部读取props，保存自己的state，执行相应的生命周期，核心原理都是闭包。
- 自定义hook的使用更加直观，高阶组件需要对组件进行嵌套，存在组件重名，可读性不好的问题。
- 高阶组件只能通过props或者context来接受外部数据，而自定义hook可以props或者其他自定义hook来接受外部输入数据，数据来源更加清晰。

### 总结

Hooks的出现给函数式组件带来了状态、生命周期的使用，通过自定义Hook，可以保存状态，读取props，执行相应的生命周期，可以对逻辑进行更细粒度的抽象，更好的复用逻辑。

