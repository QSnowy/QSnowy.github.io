Rocket.Chat项目集成

RocketChat客户端代码结构

```
.
├── ReactotronConfig.js	// 开发环境下日志配置
├── actions							// 请求动作		
├── commands.js					// 常用指令
├── constants						// 常量
├── containers					// 公用组件
├── emojis.js						// emoj 加载工具
├── i18n								// 国际化文件
├── index.js
├── lib
├── notifications				// 通知相关
├── presentation				// 
├── reducers						// 
├── sagas								//
├── selectors						//
├── share.js
├── split.js
├── static
├── tablet.js
├── theme.js
├── utils
└── views
```



#### actions

```
.actions
├── actionsTypes.js				// 接口type统一生成
├── activeUsers.js
├── connect.js
├── crashReport.js
├── createChannel.js
├── createDiscussion.js
├── customEmojis.js
├── deepLinking.js
├── index.js
├── inviteLinks.js
├── login.js
├── messages.js
├── notification.js
├── room.js
├── rooms.js
├── selectedUsers.js
├── server.js
├── settings.js
├── share.js
├── sortPreferences.js
└── usersTyping.js
```

#### contants

```
.
├── colors.js							// 主题light/dark/black颜色
├── links.js							// 应用的下载链接
├── messagesStatus.js			// 消息状态码
├── settings.js						// 用户设置、应用设置key：值类型
├── tablet.js							// 侧边栏尺寸
├── types.js							// 类型相关常量
└── userDefaults.js				// 用户相关常量键值
```

#### lib

```
.
├── Icons.js
├── ModalNavigation.js
├── Navigation.js
├── ShareNavigation.js
├── createStore.js
├── database
│   ├── index.js
│   ├── model
│   │   ├── CustomEmoji.js
│   │   ├── FrequentlyUsedEmoji.js
│   │   ├── Message.js
│   │   ├── Permission.js
│   │   ├── Role.js
│   │   ├── Room.js
│   │   ├── Server.js
│   │   ├── Setting.js
│   │   ├── SlashCommand.js
│   │   ├── Subscription.js
│   │   ├── Thread.js
│   │   ├── ThreadMessage.js
│   │   ├── Upload.js
│   │   ├── User.js
│   │   ├── migrations.js
│   │   └── serversMigrations.js
│   ├── schema
│   │   ├── app.js
│   │   └── servers.js
│   └── utils.js
├── methods
│   ├── actions.js
│   ├── callJitsi.js
│   ├── canOpenRoom.js
│   ├── getCustomEmojis.js
│   ├── getPermissions.js
│   ├── getRoles.js
│   ├── getRooms.js
│   ├── getSettings.js
│   ├── getSlashCommands.js
│   ├── getUsersPresence.js
│   ├── helpers
│   │   ├── buildMessage.js
│   │   ├── findSubscriptionsRooms.js
│   │   ├── getFileUrlFromMessage.js
│   │   ├── mergeSubscriptionsRooms.js
│   │   ├── normalizeMessage.js
│   │   ├── parseQuery.js
│   │   ├── parseUrls.js
│   │   └── protectedFunction.js
│   ├── loadMessagesForRoom.js
│   ├── loadMissedMessages.js
│   ├── loadThreadMessages.js
│   ├── logout.js
│   ├── readMessages.js
│   ├── sendFileMessage.js
│   ├── sendMessage.js
│   ├── subscriptions
│   │   ├── room.js
│   │   └── rooms.js
│   └── updateMessages.js
├── rocketchat.js
├── selection.json
└── utils.js
```

#### utils

```
.
├── avatar.js
├── debounce.js
├── deviceInfo.js
├── events.js
├── fetch.js
├── info.js
├── isReadOnly.js
├── isValidEmail.js
├── layoutAnimation.js
├── log.js
├── longPress.js
├── media.js
├── messageTypes.js
├── moment.js
├── navigation.js
├── openLink.js
├── random.js
├── review.js
├── room.js
├── scaling.js
├── scrollPersistTaps.js
├── server.js
├── shortnameToUnicode
│   ├── ascii.js
│   ├── emojis.js
│   ├── index.js
│   └── shortnameToUnicode.test.js
├── theme.js
├── throttle.js
├── touch.js
├── twoFactor.js
└── url.js
```



#### view

```
.
├── AdminPanelView								// webview展示adminInfo
├── AutoTranslateView							// 自动翻译
├── CreateDiscussionView					// 创建讨论，选择频道、选择用户
├── DirectoryView									// 频道/用户列表
├── InviteUsersEditView						// 编辑邀请用户
├── InviteUsersView								// 邀请用户
├── LanguageView									// 语言选择view
├── MessagesView								 	// 消息列表
├── NotificationPreferencesView		// 消息通知设置
├── OnboardingView								// 创建workspace引导页	
├── ProfileView										// 个人信息页面，可控制编辑权限
├── ReadReceiptView								// @信息
├── RoomActionsView								// 聊天室或私聊可操作动作列表
├── RoomInfoEditView							// 聊天室信息编辑
├── RoomInfoView									// 聊天室信息
├── RoomMembersView								// 聊天室成员列表
├── RoomView											// 聊天室详情
│   └── Header										// 自定义顶部导航栏，展示名称，未读消息
├── RoomsListView									// 聊天室/会话 列表
│   ├── Header										// 列表头部导航栏		
│   ├── ListHeader								// 聊天室/会话 列表头部组件：serachbar,sort,dirctory
│   └── SortDropdown							// 下拉排序窗口
├── SearchMessagesView						// 消息记录搜索页
├── SettingsView									// 设置页面
├── ShareListView									// 分享，
│   └── Header										// 头部菜单，含有搜索框，App内还未见到
├── ShareView											// 分享菜单，展示分享方式buttons		
├── SidebarView										// 侧边栏
├── ThreadMessagesView						// thread 消息列表
└── WorkspaceView									// 群组列表
```



