## 注册失败

在注册UA的时候添加 `contact_uri` 参数

```typescript
const configuration: UAConfiguration = {
    sockets: [socket],
    uri: `sip:${loginUser}@${host}`,
    password: password,
    display_name: displayName,
    register: true,
    authorization_user: loginUser,
    contact_uri: `sip:${loginUser}@${host};transport=wss`  // 添加这个参数
}
ua = new JsSip.UA(configuration)
```

## 打电话失败

### 422 Session Interval Too Small

在注册 UA 的时候添加 `session_timers` 参数值为 `true`

在拨打电话的时候添加 `sessionTimersExpires` 参数值为 `大于120的数字`

#### 注册UA

```typescript
const configuration: UAConfiguration = {
    sockets: [socket],
    uri: `sip:${loginUser}@${host}`,
    password: password,
    display_name: displayName,
    register: true,
    authorization_user: loginUser,
    contact_uri: `sip:${loginUser}@${host};transport=wss`,
    session_timers: true
}
ua = new JsSip.UA(configuration)
```

#### 拨打电话

```typescript
ua.call(`sip:${targetUser}@${host}`, {
    mediaConstraints: {
        audio: true,
        video: false
    },
    sessionTimersExpires: 180
})
```





