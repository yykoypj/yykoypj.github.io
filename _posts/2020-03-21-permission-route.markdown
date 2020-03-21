---
layout: post
title:  "react权限校验解决方案"
date:   2020-03-22 00:20:59 +0800
tag: [react, router, permission ]
---

# React权限校验解决方案
#### 相关的库: react, react-router v4
权限是网页开发中一个很普遍要考虑的问题

哪个路由可以被访问，哪个组件应不应该显示

react中的校验权限的思路与高阶组件的思路一致：
```javascript
function componentA() {
  return <div>hello world</div>;
}

function permissionCheck(widget) {
  // 从什么地方获取用户角色
  const userRoles = wayToGetRoles();
  // 再从什么地方获取组件需要的角色，这里仅做伪代码形式的参考
  const roleOfNeeded = getWdigetRole(widget);
  // 满足条件就返回组件，否则返回无
  if (hasPermission(userRoles, roleOfNeeded)) {
    return <Widget/>
  }
  return <NotAllow/>
}

export default permissionCheck(componentA);
```
上面的代码仅做思路参考，现实开发过程中，检查权限的过程一般放在使用的地方，而不是在组件export的时候就做硬编码，这样才能达到解耦的效果。

但是校验权限需要的基本的东西在代码里面就可以看到：
1. 获取当前登录的用户拥有的权限
2. 设定好组件需要的权限
3. 校验的方法

上面这个三要素实现的方法因各自业务系统的需求而各不相同。A系统可能可以在用户登录的时候就拿到用户所需要的所有权限，B系统则是惰性获取，**当然这里做好缓存很重要**。

这篇文章的重点是如何把上述三要素合理地组装起来，从而达到解耦和复用的目的。

首先确立一下最核心的东西
```javascript
function hasPermission(userPermission, permissionKey) {
  // 这里的userPermission和permissionKey不建议再做复杂的处理逻辑，应是最直接的数组或基础数据类型
  if(permissionKey === null || permissionKey === undefined || permissionKey === '') {
    return true;
  }
  if (!userPermission) {
    return false;
  }
  // 此处可根据需要更换逻辑
  return userPermission.includes(permissionKey);
}
```

有了上面的函数，我们就可以对判断结果做对应的处理
```javascript
// 校验的如果是路由权限
function ProtectedRoute({userPermission, permissionKey， ...rest}) {
  return (<div>
        {hasPermission(userPermission, permissionKey) ? <Route {...rest}/> : <NotAllow />}
    </div>);
}

// 如果是权限组件一般可以这么用
function ProtectedComponent({permissionKey, fallback, children}) {
  // 这里的fallback为无权限的时候的显示内容，所以使用的时候可以考虑在函数外面再包一层
  
  // 考虑到权限组件使用的便捷性，应为其提供统一的获取用户权限的方法
  // 这样就只需要在使用的时候指定组件所需permissionKey即可
  const userPermission = wayToGetUserPermission();

  return (<div>
        {hasPermission(userPermission, permissionKey) ? {children}: <fallback />}
    </div>);
}

```
权限组件的使用方式:
```javascript
<ProtectedComponent permissionKey="button1" >
  <button>有权限的人才看得到这里</button>
</ProtectedComponent>
```
而对于权限路由来说,我们明显需要做的更多:
1. 使用统一的途径生成react-router的对应配置
2. 同时生成对应的SideBar

单页应用的路由配置信息一般有两种，一种使用`config.js`配置文件，另一种使用约定式路由(参考umi)。当然如果你硬编码把`<Route {...rest}>`直接写在代码中的，建议你跳过剩下的部分..

接下来统一以`config.js`配置文件的形式讲如何编写代码，约定式路由一般有对应的权限解决方案，基本思路一致
```javascript
// config.js
export default {
  Routes: [
    {
      path: '/',
      title: '首页',
      component: IndexComponent,
    },
    {
      title: '一组页面',
      subRoutes: [
        {
          path: '/pageA',
          title: 'A页面',
          component: ComponentA,
        },
        {
          path: '/pageB',
          title: 'B页面',
          permissionKey: 'pageB'
          component: ComponentB,
        }
      ]
    }
  ]
}

// LayoutRoutes.js
import { Routes } from './config.js';
import { Route } from 'react-router-dom';
import { Menu } from 'antd';
const { Item: MenuItem, SubMenu } = Menu

function RouteSideBar() {
  // 这个函数通过遍历Routes直接生成侧边栏导航
  const routeToMenu = ({ path,  subRoutes = [], title, permissionKey }, index) => {
    const userPermission = wayToGetUserPermission();
    if (!title || !hasPermission(userPermission, permissionKey)) {
        return null
    }
    return (
        subRoutes.length ?
          <SubMenu key={path ? path + index : hash(title) + index} title={title}>
            {subRoutes.map(routeToMenu)}
          </SubMenu> 
          :
          <MenuItem key={path}>
            <Link to={path}>{title}</Link>
          </MenuItem>);
  }
  
  return <Menu>
    {Routes.map(routeToMenu)}
  </Menu>;
}

function LayoutContentRoutes() {
    // 这里通过遍历Routes生成react-router配置
    const routes = []
    const handleRoute = ({ path, subRoutes = [], permissionKey, ...rest }, index) => {
        if (subRoutes.length === 0) {
            routes.push(<ProtectedRoute permissionKey={permissionKey} key={path + index} path={path} {...rest} />)
            return
        }
        subRoutes.forEach(handleRoute)
    }

    Routes.forEach(handleRoute);
    return (
        <Switch>
            {routes}
        </Switch>
    )
}

export {
  RouteSideBar,
  LayoutContentRoutes
}
```
这样就可以在你的`router.js`文件中直接使用`<LayoutContentRoutes/>`组件，在你的布局文件中直接使用`<RouteSideBar/>`组件。

当然这里还有很多工作没有做，比如还没有Redirect功能，只有最低一级的路径才能有路由，这些都可以根据上面的代码扩展出来

最主要的是这里完成了一份相对解耦的路由方案

配置文件的修改可以直接反应到侧边栏和`react-router`的配置

你可以根据需求自定义获取用户权限的方法(比如在localstorage中存取),校验权限的方法也可以根据需要拓展使其支持其它数据类型或者异步方法。