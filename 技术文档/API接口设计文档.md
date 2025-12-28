# API接口设计文档

## 1. 文档概述

### 1.1 文档目的
本文档详细描述了私人日记App - My Diary的API接口设计，包括Firebase服务接口（认证、数据库、存储等）和本地API接口（状态管理、数据处理等）的定义、参数、返回值、使用方法等，为开发团队提供API使用的指导和规范。

### 1.2 文档范围
本文档涵盖了My Diary App的用户认证、日记管理、标签管理、数据同步、文件存储等核心功能的API接口设计。

### 1.3 术语定义
| 术语 | 定义 |
|------|------|
| API | Application Programming Interface，应用程序编程接口 |
| Firebase | Google提供的移动和Web应用开发平台 |
| RESTful API | 基于REST架构风格的API |
| Firebase Auth | Firebase提供的用户认证服务 |
| Cloud Firestore | Firebase提供的云数据库服务 |
| Firebase Storage | Firebase提供的云存储服务 |
| JSON | JavaScript Object Notation，一种轻量级的数据交换格式 |
| JWT | JSON Web Token，一种用于认证的令牌 |
| HTTPS | 超文本传输安全协议 |
| BLoC | Business Logic Component，一种状态管理模式 |

## 2. 接口设计原则

### 2.1 设计原则
- **简洁性**：接口设计简洁明了，易于理解和使用
- **一致性**：接口命名和参数格式保持一致
- **安全性**：注重接口安全，防止未授权访问
- **性能**：优化接口性能，减少响应时间
- **可扩展性**：支持未来功能扩展
- **错误处理**：提供清晰的错误信息和错误码

### 2.2 技术选型
| 技术/服务 | 用途 |
|----------|------|
| Firebase Auth | 用户认证接口 |
| Cloud Firestore | 云数据库接口 |
| Firebase Storage | 云存储接口 |
| RESTful API | 自定义后端接口（如需） |
| BLoC/Provider | 本地状态管理接口 |

### 2.3 接口分类
| 接口类型 | 描述 |
|----------|------|
| 认证接口 | 用户注册、登录、退出登录等 |
| 日记管理接口 | 日记的创建、编辑、删除、查询等 |
| 标签管理接口 | 标签的创建、编辑、删除、查询等 |
| 数据同步接口 | 本地数据与云端数据的同步 |
| 文件存储接口 | 图片等文件的上传、下载、删除等 |
| 用户设置接口 | 用户偏好设置的管理 |

## 3. Firebase服务接口

### 3.1 Firebase Auth接口

#### 3.1.1 用户注册
**功能**：创建新用户账号

**接口**：`FirebaseAuth.instance.createUserWithEmailAndPassword()`

**参数**：
| 参数名 | 类型 | 必选 | 描述 |
|--------|------|------|------|
| email | String | 是 | 用户邮箱 |
| password | String | 是 | 用户密码 |

**返回值**：
```dart
UserCredential {
  user: User {
    uid: String, // 用户唯一标识符
    email: String, // 用户邮箱
    displayName: String?, // 用户显示名称
    photoURL: String?, // 用户头像URL
    emailVerified: bool, // 邮箱是否已验证
    isAnonymous: bool, // 是否为匿名用户
    metadata: UserMetadata {
      creationTime: DateTime, // 创建时间
      lastSignInTime: DateTime, // 最后登录时间
    }
  },
  credential: AuthCredential? // 认证凭证
}
```

**使用示例**：
```dart
Future<UserCredential> register(String email, String password) async {
  try {
    UserCredential userCredential = await FirebaseAuth.instance
        .createUserWithEmailAndPassword(email: email, password: password);
    return userCredential;
  } on FirebaseAuthException catch (e) {
    if (e.code == 'weak-password') {
      throw Exception('密码强度不足');
    } else if (e.code == 'email-already-in-use') {
      throw Exception('该邮箱已被注册');
    }
    throw Exception('注册失败: ${e.message}');
  } catch (e) {
    throw Exception('注册失败: $e');
  }
}
```

#### 3.1.2 用户登录
**功能**：用户登录账号

**接口**：`FirebaseAuth.instance.signInWithEmailAndPassword()`

**参数**：
| 参数名 | 类型 | 必选 | 描述 |
|--------|------|------|------|
| email | String | 是 | 用户邮箱 |
| password | String | 是 | 用户密码 |

**返回值**：
```dart
UserCredential {
  user: User {
    uid: String, // 用户唯一标识符
    email: String, // 用户邮箱
    displayName: String?, // 用户显示名称
    photoURL: String?, // 用户头像URL
    emailVerified: bool, // 邮箱是否已验证
    isAnonymous: bool, // 是否为匿名用户
    metadata: UserMetadata {
      creationTime: DateTime, // 创建时间
      lastSignInTime: DateTime, // 最后登录时间
    }
  },
  credential: AuthCredential? // 认证凭证
}
```

**使用示例**：
```dart
Future<UserCredential> login(String email, String password) async {
  try {
    UserCredential userCredential = await FirebaseAuth.instance
        .signInWithEmailAndPassword(email: email, password: password);
    return userCredential;
  } on FirebaseAuthException catch (e) {
    if (e.code == 'user-not-found') {
      throw Exception('用户不存在');
    } else if (e.code == 'wrong-password') {
      throw Exception('密码错误');
    }
    throw Exception('登录失败: ${e.message}');
  } catch (e) {
    throw Exception('登录失败: $e');
  }
}
```

#### 3.1.3 用户退出登录
**功能**：用户退出当前账号

**接口**：`FirebaseAuth.instance.signOut()`

**参数**：无

**返回值**：`Future<void>`

**使用示例**：
```dart
Future<void> logout() async {
  try {
    await FirebaseAuth.instance.signOut();
  } catch (e) {
    throw Exception('退出登录失败: $e');
  }
}
```

#### 3.1.4 重置密码
**功能**：发送密码重置邮件

**接口**：`FirebaseAuth.instance.sendPasswordResetEmail()`

**参数**：
| 参数名 | 类型 | 必选 | 描述 |
|--------|------|------|------|
| email | String | 是 | 用户邮箱 |

**返回值**：`Future<void>`

**使用示例**：
```dart
Future<void> resetPassword(String email) async {
  try {
    await FirebaseAuth.instance.sendPasswordResetEmail(email: email);
  } on FirebaseAuthException catch (e) {
    if (e.code == 'user-not-found') {
      throw Exception('用户不存在');
    }
    throw Exception('发送密码重置邮件失败: ${e.message}');
  } catch (e) {
    throw Exception('发送密码重置邮件失败: $e');
  }
}
```

#### 3.1.5 获取当前用户
**功能**：获取当前登录用户信息

**接口**：`FirebaseAuth.instance.currentUser`

**参数**：无

**返回值**：`User?`

**使用示例**：
```dart
User? getCurrentUser() {
  return FirebaseAuth.instance.currentUser;
}
```

### 3.2 Cloud Firestore接口

#### 3.2.1 获取日记列表
**功能**：获取用户的日记列表

**接口**：`FirebaseFirestore.instance.collection('diaries').where().get()`

**参数**：
| 参数名 | 类型 | 必选 | 描述 |
|--------|------|------|------|
| userId | String | 是 | 用户ID |
| limit | int | 否 | 返回结果数量限制 |
| orderBy | String | 否 | 排序字段 |
| orderDirection | bool | 否 | 排序方向(true:升序, false:降序) |

**返回值**：
```dart
QuerySnapshot<Map<String, dynamic>> {
  docs: List<QueryDocumentSnapshot<Map<String, dynamic>>> {
    QueryDocumentSnapshot<Map<String, dynamic>> {
      id: String, // 文档ID
      data(): Map<String, dynamic> {
        user_id: String, // 用户ID
        title: String, // 日记标题
        content: String, // 日记内容
        mood: int, // 心情
        weather: int, // 天气
        location: String?, // 位置
        tags: List<String>, // 标签ID列表
        images: List<String>, // 图片路径列表
        created_at: Timestamp, // 创建时间
        updated_at: Timestamp, // 更新时间
        is_deleted: bool, // 是否删除
      }
    }
  }
}
```

**使用示例**：
```dart
Future<List<Diary>> getDiaryList(String userId, {int limit = 20}) async {
  try {
    QuerySnapshot<Map<String, dynamic>> snapshot = await FirebaseFirestore.instance
        .collection('diaries')
        .where('user_id', isEqualTo: userId)
        .where('is_deleted', isEqualTo: false)
        .orderBy('created_at', descending: true)
        .limit(limit)
        .get();
    
    return snapshot.docs.map((doc) => Diary.fromFirestore(doc)).toList();
  } catch (e) {
    throw Exception('获取日记列表失败: $e');
  }
}
```

#### 3.2.2 创建日记
**功能**：创建新日记

**接口**：`FirebaseFirestore.instance.collection('diaries').add()`

**参数**：
| 参数名 | 类型 | 必选 | 描述 |
|--------|------|------|------|
| diary | Map<String, dynamic> | 是 | 日记数据 |

**返回值**：
```dart
DocumentReference<Map<String, dynamic>> {
  id: String, // 新创建的文档ID
  path: String, // 文档路径
}
```

**使用示例**：
```dart
Future<DocumentReference> createDiary(Diary diary) async {
  try {
    return await FirebaseFirestore.instance.collection('diaries').add(diary.toFirestore());
  } catch (e) {
    throw Exception('创建日记失败: $e');
  }
}
```

#### 3.2.3 更新日记
**功能**：更新日记内容

**接口**：`FirebaseFirestore.instance.collection('diaries').doc().update()`

**参数**：
| 参数名 | 类型 | 必选 | 描述 |
|--------|------|------|------|
| diaryId | String | 是 | 日记ID |
| diary | Map<String, dynamic> | 是 | 更新后的日记数据 |

**返回值**：`Future<void>`

**使用示例**：
```dart
Future<void> updateDiary(String diaryId, Diary diary) async {
  try {
    await FirebaseFirestore.instance.collection('diaries').doc(diaryId).update(diary.toFirestore());
  } catch (e) {
    throw Exception('更新日记失败: $e');
  }
}
```

#### 3.2.4 删除日记
**功能**：删除日记（软删除）

**接口**：`FirebaseFirestore.instance.collection('diaries').doc().update()`

**参数**：
| 参数名 | 类型 | 必选 | 描述 |
|--------|------|------|------|
| diaryId | String | 是 | 日记ID |

**返回值**：`Future<void>`

**使用示例**：
```dart
Future<void> deleteDiary(String diaryId) async {
  try {
    await FirebaseFirestore.instance.collection('diaries').doc(diaryId).update({
      'is_deleted': true,
      'updated_at': FieldValue.serverTimestamp(),
    });
  } catch (e) {
    throw Exception('删除日记失败: $e');
  }
}
```

#### 3.2.5 获取标签列表
**功能**：获取用户的标签列表

**接口**：`FirebaseFirestore.instance.collection('tags').where().get()`

**参数**：
| 参数名 | 类型 | 必选 | 描述 |
|--------|------|------|------|
| userId | String | 是 | 用户ID |

**返回值**：
```dart
QuerySnapshot<Map<String, dynamic>> {
  docs: List<QueryDocumentSnapshot<Map<String, dynamic>>> {
    QueryDocumentSnapshot<Map<String, dynamic>> {
      id: String, // 文档ID
      data(): Map<String, dynamic> {
        user_id: String, // 用户ID
        name: String, // 标签名称
        color: String, // 标签颜色
        created_at: Timestamp, // 创建时间
        updated_at: Timestamp, // 更新时间
        is_deleted: bool, // 是否删除
      }
    }
  }
}
```

**使用示例**：
```dart
Future<List<Tag>> getTagList(String userId) async {
  try {
    QuerySnapshot<Map<String, dynamic>> snapshot = await FirebaseFirestore.instance
        .collection('tags')
        .where('user_id', isEqualTo: userId)
        .where('is_deleted', isEqualTo: false)
        .orderBy('created_at', descending: true)
        .get();
    
    return snapshot.docs.map((doc) => Tag.fromFirestore(doc)).toList();
  } catch (e) {
    throw Exception('获取标签列表失败: $e');
  }
}
```

### 3.3 Firebase Storage接口

#### 3.3.1 上传图片
**功能**：上传图片到云存储

**接口**：`FirebaseStorage.instance.ref().child().putFile()`

**参数**：
| 参数名 | 类型 | 必选 | 描述 |
|--------|------|------|------|
| file | File | 是 | 图片文件 |
| userId | String | 是 | 用户ID |
| diaryId | String | 是 | 日记ID |

**返回值**：
```dart
UploadTask {
  snapshotEvents: Stream<TaskSnapshot>, // 上传进度流
  snapshot: TaskSnapshot?, // 当前上传快照
  pause(): void, // 暂停上传
  resume(): void, // 恢复上传
  cancel(): Future<void>, // 取消上传
  then(): Future<TaskSnapshot>, // 上传完成回调
}
```

**使用示例**：
```dart
Future<String> uploadImage(File file, String userId, String diaryId) async {
  try {
    String fileName = '${DateTime.now().millisecondsSinceEpoch}.jpg';
    Reference ref = FirebaseStorage.instance
        .ref()
        .child('images')
        .child(userId)
        .child(diaryId)
        .child(fileName);
    
    UploadTask uploadTask = ref.putFile(file);
    TaskSnapshot snapshot = await uploadTask;
    String downloadUrl = await snapshot.ref.getDownloadURL();
    
    return downloadUrl;
  } catch (e) {
    throw Exception('上传图片失败: $e');
  }
}
```

#### 3.3.2 下载图片
**功能**：从云存储下载图片

**接口**：`FirebaseStorage.instance.ref().child().getData()`

**参数**：
| 参数名 | 类型 | 必选 | 描述 |
|--------|------|------|------|
| imagePath | String | 是 | 图片路径 |
| maxSize | int | 否 | 最大下载大小(字节) |

**返回值**：`Future<Uint8List>`

**使用示例**：
```dart
Future<Uint8List> downloadImage(String imagePath) async {
  try {
    Reference ref = FirebaseStorage.instance.ref().child(imagePath);
    return await ref.getData(10 * 1024 * 1024); // 最大10MB
  } catch (e) {
    throw Exception('下载图片失败: $e');
  }
}
```

#### 3.3.3 删除图片
**功能**：从云存储删除图片

**接口**：`FirebaseStorage.instance.ref().child().delete()`

**参数**：
| 参数名 | 类型 | 必选 | 描述 |
|--------|------|------|------|
| imagePath | String | 是 | 图片路径 |

**返回值**：`Future<void>`

**使用示例**：
```dart
Future<void> deleteImage(String imagePath) async {
  try {
    Reference ref = FirebaseStorage.instance.ref().child(imagePath);
    await ref.delete();
  } catch (e) {
    throw Exception('删除图片失败: $e');
  }
}
```

## 4. 本地API接口

### 4.1 本地数据库接口

#### 4.1.1 初始化数据库
**功能**：初始化本地SQLite数据库

**接口**：`DatabaseHelper.instance.initDatabase()`

**参数**：无

**返回值**：`Future<Database>`

**使用示例**：
```dart
Future<void> initializeDatabase() async {
  Database database = await DatabaseHelper.instance.initDatabase();
  // 使用数据库
}
```

#### 4.1.2 获取本地日记列表
**功能**：获取本地存储的日记列表

**接口**：`DatabaseHelper.instance.getDiaryList()`

**参数**：
| 参数名 | 类型 | 必选 | 描述 |
|--------|------|------|------|
| userId | String | 是 | 用户ID |
| limit | int | 否 | 返回结果数量限制 |

**返回值**：`Future<List<Diary>>`

**使用示例**：
```dart
Future<List<Diary>> getLocalDiaryList(String userId) async {
  return await DatabaseHelper.instance.getDiaryList(userId, limit: 50);
}
```

### 4.2 状态管理接口

#### 4.2.1 用户认证状态管理
**功能**：管理用户认证状态

**接口**：`AuthBloc`

**事件**：
- `AuthEvent.login()`：登录事件
- `AuthEvent.logout()`：退出登录事件
- `AuthEvent.checkStatus()`：检查认证状态事件

**状态**：
- `AuthState.initial()`：初始状态
- `AuthState.loading()`：加载状态
- `AuthState.authenticated()`：已认证状态
- `AuthState.unauthenticated()`：未认证状态
- `AuthState.error()`：错误状态

**使用示例**：
```dart
// 登录
context.read<AuthBloc>().add(AuthEvent.login(email: 'user@example.com', password: 'password'));

// 监听认证状态变化
BlocBuilder<AuthBloc, AuthState>(
  builder: (context, state) {
    if (state is AuthState.authenticated) {
      return HomePage();
    } else if (state is AuthState.unauthenticated) {
      return LoginPage();
    } else {
      return LoadingPage();
    }
  },
)
```

#### 4.2.2 日记状态管理
**功能**：管理日记数据状态

**接口**：`DiaryBloc`

**事件**：
- `DiaryEvent.fetchDiaries()`：获取日记列表事件
- `DiaryEvent.createDiary()`：创建日记事件
- `DiaryEvent.updateDiary()`：更新日记事件
- `DiaryEvent.deleteDiary()`：删除日记事件

**状态**：
- `DiaryState.initial()`：初始状态
- `DiaryState.loading()`：加载状态
- `DiaryState.success()`：成功状态
- `DiaryState.error()`：错误状态

**使用示例**：
```dart
// 获取日记列表
context.read<DiaryBloc>().add(DiaryEvent.fetchDiaries());

// 监听日记状态变化
BlocBuilder<DiaryBloc, DiaryState>(
  builder: (context, state) {
    if (state is DiaryState.success) {
      return DiaryListWidget(diaries: state.diaries);
    } else if (state is DiaryState.loading) {
      return LoadingWidget();
    } else {
      return ErrorWidget(message: state.message);
    }
  },
)
```

### 4.3 加密解密接口

#### 4.3.1 加密数据
**功能**：加密敏感数据

**接口**：`EncryptionService.encrypt()`

**参数**：
| 参数名 | 类型 | 必选 | 描述 |
|--------|------|------|------|
| data | String | 是 | 待加密的数据 |
| key | String | 是 | 加密密钥 |

**返回值**：`String`（加密后的数据）

**使用示例**：
```dart
String encryptedData = await EncryptionService.encrypt('敏感数据', '加密密钥');
```

#### 4.3.2 解密数据
**功能**：解密敏感数据

**接口**：`EncryptionService.decrypt()`

**参数**：
| 参数名 | 类型 | 必选 | 描述 |
|--------|------|------|------|
| encryptedData | String | 是 | 加密后的数据 |
| key | String | 是 | 解密密钥 |

**返回值**：`String`（解密后的数据）

**使用示例**：
```dart
String decryptedData = await EncryptionService.decrypt(encryptedData, '加密密钥');
```

## 5. 错误码设计

### 5.1 Firebase错误码

#### 5.1.1 认证错误码
| 错误码 | 描述 |
|--------|------|
| `invalid-email` | 邮箱格式无效 |
| `user-disabled` | 用户账号已被禁用 |
| `user-not-found` | 用户不存在 |
| `wrong-password` | 密码错误 |
| `email-already-in-use` | 邮箱已被注册 |
| `weak-password` | 密码强度不足 |
| `operation-not-allowed` | 操作不被允许 |

#### 5.1.2 Firestore错误码
| 错误码 | 描述 |
|--------|------|
| `permission-denied` | 权限被拒绝 |
| `not-found` | 文档或集合不存在 |
| `already-exists` | 文档已存在 |
| `invalid-argument` | 参数无效 |
| `deadline-exceeded` | 请求超时 |
| `unavailable` | 服务不可用 |

### 5.2 自定义错误码

| 错误码 | 描述 |
|--------|------|
| `APP-001` | 应用初始化失败 |
| `APP-002` | 数据库连接失败 |
| `APP-003` | 数据加密失败 |
| `APP-004` | 数据解密失败 |
| `APP-005` | 图片上传失败 |
| `APP-006` | 图片下载失败 |
| `APP-007` | 数据同步失败 |
| `APP-008` | 生物识别认证失败 |
| `APP-009` | 应用锁验证失败 |
| `APP-010` | 网络连接失败 |

## 6. 接口安全设计

### 6.1 认证与授权
- **用户认证**：使用Firebase Auth进行用户认证，支持邮箱密码认证和第三方登录
- **JWT令牌**：使用JWT令牌进行API调用认证
- **权限控制**：Firebase Security Rules控制数据访问权限，用户只能访问自己的数据
- **会话管理**：实现会话超时机制，增强安全性

### 6.2 数据加密
- **传输加密**：所有API请求使用HTTPS协议
- **存储加密**：敏感数据（如日记内容）在本地加密后再存储到本地数据库和云端
- **密钥管理**：使用flutter_secure_storage安全存储加密密钥

### 6.3 防攻击措施
- **输入验证**：对所有用户输入进行验证，防止SQL注入和XSS攻击
- **请求限制**：限制API请求频率，防止暴力攻击
- **错误信息**：避免在错误信息中泄露敏感信息
- **日志记录**：记录关键操作日志，便于安全审计

### 6.4 隐私保护
- **数据最小化**：仅收集必要的用户数据
- **隐私政策**：提供透明的隐私政策，告知用户数据收集和使用方式
- **数据删除**：支持用户删除自己的数据
- **GDPR合规**：遵循GDPR和其他隐私法规

## 7. 接口测试策略

### 7.1 测试类型
- **单元测试**：测试单个API接口的功能
- **集成测试**：测试API接口之间的集成
- **端到端测试**：测试完整的API调用流程
- **性能测试**：测试API接口的性能和响应时间
- **安全测试**：测试API接口的安全性

### 7.2 测试工具
- **Flutter Test**：单元测试和集成测试
- **Postman**：API接口测试
- **Firebase Test Lab**：移动应用测试
- **Jmeter**：性能测试

### 7.3 测试环境
- **开发环境**：开发过程中使用的测试环境
- **测试环境**：专门用于测试的环境
- **预发布环境**：接近生产环境的测试环境
- **生产环境**：实际用户使用的环境

## 8. 附录

### 8.1 API接口清单

| 接口类型 | 接口名称 | 功能描述 |
|----------|----------|----------|
| 认证接口 | 用户注册 | 创建新用户账号 |
| 认证接口 | 用户登录 | 用户登录账号 |
| 认证接口 | 退出登录 | 用户退出当前账号 |
| 认证接口 | 重置密码 | 发送密码重置邮件 |
| 认证接口 | 获取当前用户 | 获取当前登录用户信息 |
| 日记接口 | 获取日记列表 | 获取用户的日记列表 |
| 日记接口 | 创建日记 | 创建新日记 |
| 日记接口 | 更新日记 | 更新日记内容 |
| 日记接口 | 删除日记 | 删除日记 |
| 标签接口 | 获取标签列表 | 获取用户的标签列表 |
| 标签接口 | 创建标签 | 创建新标签 |
| 标签接口 | 更新标签 | 更新标签信息 |
| 标签接口 | 删除标签 | 删除标签 |
| 存储接口 | 上传图片 | 上传图片到云存储 |
| 存储接口 | 下载图片 | 从云存储下载图片 |
| 存储接口 | 删除图片 | 从云存储删除图片 |
| 同步接口 | 数据同步 | 同步本地数据与云端数据 |

### 8.2 数据模型定义

#### 8.2.1 日记模型
```dart
class Diary {
  final String? id;
  final String userId;
  final String title;
  final String content;
  final int? mood;
  final int? weather;
  final String? location;
  final List<String> tags;
  final List<String> images;
  final DateTime createdAt;
  final DateTime updatedAt;
  final bool isDeleted;
  
  Diary({
    this.id,
    required this.userId,
    required this.title,
    required this.content,
    this.mood,
    this.weather,
    this.location,
    this.tags = const [],
    this.images = const [],
    required this.createdAt,
    required this.updatedAt,
    this.isDeleted = false,
  });
  
  // 从Firestore数据转换
  factory Diary.fromFirestore(QueryDocumentSnapshot doc) {
    final data = doc.data() as Map<String, dynamic>;
    return Diary(
      id: doc.id,
      userId: data['user_id'],
      title: data['title'],
      content: data['content'],
      mood: data['mood'],
      weather: data['weather'],
      location: data['location'],
      tags: List<String>.from(data['tags'] ?? []),
      images: List<String>.from(data['images'] ?? []),
      createdAt: (data['created_at'] as Timestamp).toDate(),
      updatedAt: (data['updated_at'] as Timestamp).toDate(),
      isDeleted: data['is_deleted'] ?? false,
    );
  }
  
  // 转换为Firestore数据
  Map<String, dynamic> toFirestore() {
    return {
      'user_id': userId,
      'title': title,
      'content': content,
      'mood': mood,
      'weather': weather,
      'location': location,
      'tags': tags,
      'images': images,
      'created_at': Timestamp.fromDate(createdAt),
      'updated_at': Timestamp.fromDate(updatedAt),
      'is_deleted': isDeleted,
    };
  }
}
```

#### 8.2.2 标签模型
```dart
class Tag {
  final String? id;
  final String userId;
  final String name;
  final String color;
  final DateTime createdAt;
  final DateTime updatedAt;
  final bool isDeleted;
  
  Tag({
    this.id,
    required this.userId,
    required this.name,
    required this.color,
    required this.createdAt,
    required this.updatedAt,
    this.isDeleted = false,
  });
  
  // 从Firestore数据转换
  factory Tag.fromFirestore(QueryDocumentSnapshot doc) {
    final data = doc.data() as Map<String, dynamic>;
    return Tag(
      id: doc.id,
      userId: data['user_id'],
      name: data['name'],
      color: data['color'],
      createdAt: (data['created_at'] as Timestamp).toDate(),
      updatedAt: (data['updated_at'] as Timestamp).toDate(),
      isDeleted: data['is_deleted'] ?? false,
    );
  }
  
  // 转换为Firestore数据
  Map<String, dynamic> toFirestore() {
    return {
      'user_id': userId,
      'name': name,
      'color': color,
      'created_at': Timestamp.fromDate(createdAt),
      'updated_at': Timestamp.fromDate(updatedAt),
      'is_deleted': isDeleted,
    };
  }
}
```

---

**文档版本**：v1.0
**创建日期**：2025年12月28日
**审核人**：[待填写]
**批准人**：[待填写]