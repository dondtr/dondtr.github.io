---
title: "JWT attack"
date: 2026-10-06 00:00:00 +0700
categories: [Web Security, JWT]
tags: [jwt, web, authentication, portswigger, nodejs]
media_subpath: /assets/img/posts/jwt-attack/
---

![image.png](image.png)

## Jwt là gì?

JWT là một token chứa thông tin dưới định dạng JSON được chuyển mã (Base64Url), được ký số để chống giả mạo, và thường được dùng để xác thực cũng như ủy quyền người dùng trong các hệ thống webapp

## Cấu trúc của JSON Web Token:

![image.png](image-1.png)

Jwt có cấu trúc gồm 3 phần và được phân cách bởi dấu “.”. 

Trong đó phần header là một đối tượng json chứa các metadata. Trong đó có các trường dữ liệu như “alg” dùng để xác định thuật toán mã và được sử dụng để sign, verify token. Một số thuật toán điển hình là “HS256”, “RS256”,.. . Ngoài ra còn có trường “typ” dùng để xác định loại token và thường được sử dụng nhiều nhất là “jwt”.

```json
{
  "alg": "HS256",
  "typ": "JWT",
  "kid": "vullab-rs256-key-1"
}
```

Payload cũng là một đối tượng json chứa các claims. Claims là một các biểu thức về một thực thể (chẳng hạn user) và một số metadata phụ trợ. Có 3 loại claims thường gặp trong Payload: reserved, public và private claims.

Reserved claims: Đây là một số metadata được định nghĩa trước, trong đó một số metadata là bắt buộc, số còn lại nên tuân theo để JWT hợp lệ và đầy đủ thông tin: iss (issuer), iat (issued-at time) exp (expiration time), sub (subject), aud (audience), jti (Unique Identifier cho JWT, Can be used to prevent the JWT from being replayed. This is helpful for a one time use token.) ... Ví dụ:

```json
{
  "id": "60751034-90d3-4018-a8f7-63eef8e3ca60",
  "username": "carlos",
  "role": "user",
  "createdAt": "2026-02-22T15:09:06.718Z",
  "iat": 1773116401,
  "exp": 1773120001,
  "aud": "vullab-client",
  "iss": "vullab-api"
}
```

Public Claims - Claims được cộng đồng công nhận và sử dụng rộng rãi. Private Claims - Claims tự định nghĩa (không được trùng với Reserved Claims và Public Claims), được tạo ra để chia sẻ thông tin giữa 2 parties đã thỏa thuận và thống nhất trước đó.

Chữ ký Signature trong JWT là một chuỗi được mã hóa bởi header, payload cùng với một chuỗi bí mật theo nguyên tắc sau:

```jsx
HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
```

Do bản thân Signature đã bao gồm cả header và payload nên Signature có thể dùng để kiểm tra tính toàn vẹn của dữ liệu khi truyền tải.

## Cách sử dụng

**Xác thực (Authentication):** Khi user đăng nhập thành công bằng thông tin xác thực thì một ID token (mã thông báo định danh) sẽ được trả về.

**Ủy quyền (Authorization):** Khi user xác thực thành công và muốn truy cập và truy xuất dữ liệu thì lúc này trong mỗi request phải được đính kèm theo Access Token để xác đinh quyền hạn của user đối với tài nguyên mà user yêu cầu

**Trao đổi thông tin (Information exchange):** Vì jwt có thể được sign do đó an toàn khi dùng để trao đổi giữa các bên. Có thể xác minh được ngay nếu token có bị thay đổi dù chỉ một bit.

## Sự khác biệt giữa JWT vs Session Cookie

**1. Chữ ký mã hóa**

JWT được ký bằng chữ ký mã hóa (cryptographic signature) nhằm đảm bảo tính toàn vẹn và xác thực của dữ liệu. Điều này có nghĩa là nếu dữ liệu trong token bị thay đổi, chữ ký sẽ không còn hợp lệ. Trong khi đó, session cookie không có cơ chế ký tương tự, nên mức độ đảm bảo an toàn thấp hơn trong một số tình huống.

**2. JSON không lưu trạng thái**

JWT hoạt động theo nguyên tắc stateless – không lưu trạng thái phía server. Tất cả thông tin xác thực  đều được lưu ở phía client, giúp giảm thiểu việc truy vấn vào db mỗi lần xác thực. Nhờ đó, hệ thống tiết kiệm tài nguyên và cho phép người dùng được xác thực nhiều lần mà không cần liên tục liên hệ với server. Cookie hoạt động theo nguyên tắc statefull phía client chỉ lưu session ID. Còn dữ liệu trong phiên làm việc của user thì được lưu lại phía server.

**3. Khả năng mở rộng**

Session cookie lưu thông tin xác thực trong bộ nhớ của server, điều này khiến nó tiêu tốn nhiều tài nguyên khi số lượng người dùng tăng cao. Ngược lại, vì JWT là stateless và xử lý phần lớn ở phía client, nó không phụ thuộc nhiều vào server, giúp hệ thống dễ mở rộng hơn khi phục vụ số lượng lớn người dùng.

**4. Xác thực trên nhiều địa điểm**

Session cookie chỉ hoạt động hiệu quả trong một domain hoặc các subdomain của nó. Khi cần xác thực giữa các hệ thống khác nhau như nhiều domain hoặc giữa web và app, cookie sẽ gặp hạn chế do trình duyệt chặn. JWT thì ngược lại – được lưu trong header của request và hoạt động độc lập với domain, cho phép xác thực xuyên suốt các hệ thống như web, mobile, [API](https://vietnix.vn/api-la-gi/) một cách linh hoạt và an toàn.

Ref:

- [Tìm Hiểu Về Json Web Token (JWT) - Viblo](https://viblo.asia/p/tim-hieu-ve-json-web-token-jwt-7rVRqp73v4bP)
- [jwt header-The Complete Guide to json web token Metadata - json web token](https://jwt.json-format.com/blog/mastering-the-jwt-header-a-comprehensive-how-to-guide/)
- [JSON Web Tokens - Auth0 Docs](https://auth0.com/docs/secure/tokens/json-web-tokens#usage)
- [https://portswigger.net/web-security/jwt](https://portswigger.net/web-security/jwt)
- [Sự khác nhau giữa JWT vs Session Cookie chi tiết nhất](https://vietnix.vn/jwt-vs-session/)

## JWT Attack

***Ở đây do là có nhiều wu về lab trên portswigger rồi nên sẽ không trình bày lại thay vào đó thì mình sẽ dựng lại lab riêng cho từng lỗ hổng và chỉ rõ rootcause và phân tích source code, đi kèm là bản vá của lỗ hổng, maybe sẽ phân tích thêm và lấy ở các CVE đã công bố.***

### Lab: JWT authentication bypass via unverified signature

![image.png](image-2.png)

WebApp được xây dựng bằng nodejs và sử dụng express, postgresql, redis,.. Do đang trong quá trình hoàn thiện nên mình sẽ không public src. Mình chỉ show flow hoạt động và một phần của codebase. Để mô phỏng lỗ hổng liên quan tới jwt thì mình sử dụng [jsonwebtoken - npm](https://www.npmjs.com/package/jsonwebtoken) 

Sẽ có 2 api chính là */api/v1/jwt/unverified-signature/login* và  */api/v1/jwt/unverified-signature/profile.* Cái đầu dùng để login với credential và nhận access token. Còn api còn lại là để lấy thông tin của user.

```jsx
//src\routes\v1\jwt.route.js
router.post(
  "/unverified-signature/login",
  loginRules,
  handleValidation,
  jwtController.loginUnverifiedSignature,
);
```

Route này có 2 middleware để check xem thông tin user gửi lên có đúng định dạng hay không và để format respone trả về nếu có lỗi. Nên mình sẽ không trình bày ở đây.

```jsx
  //src\controllers\v1\jwt.controller.js
  async loginUnverifiedSignature(req, res, next) {
    try {
      const user = await jwtService.validateCredentials(req.body);
      const token = generateAccessToken(user);
      return successResponse(res, token, "Login successfully");
    } catch (error) {
      next(error);
    }
  },
```

Controller *loginUnverifiedSignature* đẩy request body cho *validateCredentials* để verify user. Sau đó sẽ tạo access token và trả về trong respone body. Ở đây các route và xác lab sau đó sẽ tái sử dụng lại *validateCredentials* để verify user credential

```jsx
  //src\services\v1\jwt.service.js
  async validateCredentials(data) {
    const { password, username } = data;
    const existedUser = await User.findOne({ where: { username: username } });

    const salt = await bcrypt.genSalt(BCRYPT_SALT_ROUNDS);
    const dummyHash = await bcrypt.hash(BCRYPT_DUMMY_PASSWORD, salt);
    const targetHash = existedUser ? existedUser.password : dummyHash;

    const isMatch = await bcrypt.compare(password, targetHash);

    if (!existedUser || !isMatch) {
      throw new AppError(401, "Invalid username or password");
    }

    return {
      id: existedUser.id,
      username: existedUser.username,
      role: existedUser.role,
      createdAt: existedUser.createdAt,
    };
  },
```

Service nhận username và password để verify user. Mục đích của mình ở đây khi sử dụng dummyHash là để tránh lỗ hổng user enumerate timing attack tham khảo tại [đây](https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-response-timing) nhé và [đây](https://github.com/spring-projects/spring-security/blob/c5632ccd838fcb2753a978918561081cff037510/core/src/main/java/org/springframework/security/authentication/dao/DaoAuthenticationProvider.java#L145)

```jsx
//src\utils\jwt.js
/*
const accessTokenOptions = {
  expiresIn: JWT_ACCESS_EXPIRES_IN,
  issuer: JWT_ISSUER,
  audience: JWT_AUDIENCE,
  algorithm: "HS256",
};
*/
const generateAccessToken = (payload) => {
  return jwt.sign(payload, JWT_SECRET, accessTokenOptions) 
```

generateAccessToken để sign jwt, sử dụng alg là HS256 và các field khác chỗ mình comment lại ấy.
Nếu không có lỗi thì response trả về là access token trong trường data.

![image.png](image-3.png)

Phía trên là req và res khi login thành công. Trường data là jwt access token. 

```jsx
//src\routes\v1\jwt.route.js
router.get(
  "/unverified-signature/profile",
  requireAuthJwtButUnverifiedSignature,
  profileController.getProfile,
);
```

Route profile có middleware *requireAuthJwtButUnverifiedSignature* cũng chính là nơi bị lỗ hổng 

```jsx
//src\middlewares\auth-jwt.middleware.js
const requireAuthJwtButUnverifiedSignature = async (req, res, next) => {
  const token = req.get("Authorization")?.split(" ")[1];
  if (!token) {
    return next(new AppError(401, "Unauthorized"));
  }

  const payload = decodeToken(token);

  if (payload?.username) {
    //Use the same username field as portswigger lab
    try {
      const user = await userService.getUserByUsername(payload.username);
      if (!user) {
        return next(new AppError(401, "Unauthorized"));
      }
      req.authUserId = user.id;
      return next();
    } catch {
      return next(new AppError(401, "Unauthorized"));
    }
  }

  if (payload?.id) {
    req.authUserId = payload.id;
    return next();
  }
  return next(new AppError(401, "Unauthorized"));
};
```

Middleware này chịu trách nghiệm kiểm tra xem có Authorization header hay không. Nếu có thì sẽ lấy ra token từ header và lấy userId từ payload token. Ở đây logic có vẽ hơi rắm vì cơ bản mục đích mình viết như này để cho giống trong lab. Lab nhận field username để lấy profile của user. Mình làm y chang và bổ sung thêm trường hợp không có username thì thay thế bằng user id.

```jsx
//src\utils\jwt.js
const decodeToken = (token) => {
  return jwt.decode(token); // jwt.decode never throws, returns null on invalid input
};
```

Về cơ bản đây chính là nơi bị vul :))). Cơ bản thì không có gì phức tạp cả. 

![image.png](image-4.png)

Trong ảnh là nội dung trong [docs](https://www.npmjs.com/package/jsonwebtoken?activeTab=readme) của jsonwebtoken. Nó cũng nói rõ là hàm sẽ chỉ “decode” chứ không “verify”. Nếu không có options complete thì kết quả là phần payload trong jwt sau khi decode. Kỹ hơn thì phân tích src code dưới đây nhé.

![image.png](image-5.png)

[Xem](https://github.com/auth0/node-jsonwebtoken/blob/master/decode.js) đoạn code trên thì jwt token được truyền vào jws.decode thuộc thư viện jws và sau khi có kết quả thì payload trong jwt sẽ được parse và return, nếu có thêm options complete thì return toàn bộ header, payload và signature. Do hàm sử dụng jws.decode nên sẽ phân tích nó làm gì.

![jwt1.2.png](jwt1.2.png)

Bạn có thể tự vô [link](https://github.com/auth0/node-jws/blob/master/lib/verify-stream.js) và đọc nhé. Code mình lấy từ thư viện jws. jws.decode được gọi thì jwsDecode sẽ được thực thi. Sau khi chuẩn hóa và kiểm tra định dạng thì jwt sẽ được bọc tách, hàm headerFromJWS sẽ lấy phần đầu của jwt tức là header decode base64 thành binary và được prase json chuyển đổi thành một đối tượng js. Tiếp đến là lấy ra payload, payload được hàm payloadFromJWS decode base64 và được prase thành js obj nếu typ là “JWT” hay có options là json còn không thì dữ nguyên. Signature cũng được phân tách và giữ nguyên. Kết quả là đối tượng chứa đầy đủ 3 thành phần đã được phân tách.

Sau một hồi quẩn đi quẩn lại thì về bản chất hàm chỉ decode base64 cho header và payload và không có một cơ chế verify nào, không kiểm tra xem signature có hợp lệ không và jwt có bị modify hay không. :)))

```jsx
//src\controllers\v1\profile.controller.js
const profileController = {
  async getProfile(req, res, next) {
    try {
      const result = await userService.getUserById(req.authUserId);
      return successResponse(res, result, "Profile fetched successfully");
    } catch (error) {
      next(error);
    }
  },
};
```

Quay trở lại với flow thì sau khi có được user id từ phần payload thì sẽ được dùng để lấy user data
và kết quả sẽ được trả về nếu không có lỗi xảy ra.

```jsx
//src\services\v1\user.service.js
const userService = {
  async getUserById(id) {
    return await User.findByPk(id, {
      attributes: { exclude: ["password"] },
    });
  },
```

Trên là userService với getUserById thực hiện chức năng truy vấn cơ cở dữ liệu để lấy ra User bằng khóa chính là user id. 

![image.png](image-6.png)

Đây là request và response sau khi gửi kèm jwt token tới */profile*

![jwt1.png](jwt1.png)

Trên là hỉnh ảnh khai thác lỗ hổng sau khi sử dụng json web tokens của burpsuite để modify trường *username* thành *vul* trong payload của jwt. Response trả về là dữ liệu của user có *username* là vul.

### Lab: JWT authentication bypass via flawed signature verification

![image.png](image-7.png)

Ở lab này cũng có 2 route là */api/v1/jwt/flawed-signature-verification/login* và */api/v1/jwt/flawed-signature-verification/profile.* login được xây dựng giống lab trên để verify user credential và tạo jwt token.  */api/v1/jwt/flawed-signature-verification/profile* Để lấy user data. Để tránh dài dòng nên mình sẽ chỉ show ra những chỗ flow chính còn code tái sử dụng thì sẽ nhắc lại thay vì show lại.

![image.png](image-8.png)

Trên là req, res tại /profile y chang lab trên. Vul chính ở route sau

```jsx
//src\routes\v1\jwt.route.js
router.get(
  "/flawed-signature-verification/profile",
  requireAuthJwtButFlawedSignatureVerification,
  profileController.getProfile,
);
```

route profile cũng có 2 middlware. Cần tập trung vào *requireAuthJwtButFlawedSignatureVerification* còn getProfile bạn xem lại code ở lab trên nhé. 

```jsx
//src\middlewares\auth-jwt.middleware.js
const requireAuthJwtButFlawedSignatureVerification = async (req, res, next) => {
  const token = req.get("Authorization")?.split(" ")[1];
  if (!token) {
    return next(new AppError(401, "Unauthorized"));
  }

  const payload = verifyUnsignedAccessToken(token);

  if (payload?.username) {
    //Use the same username field as portswigger lab
    try {
      const user = await userService.getUserByUsername(payload.username);
      if (!user) {
        return next(new AppError(401, "Unauthorized"));
      }
      req.authUserId = user.id;
      return next();
    } catch {
      return next(new AppError(401, "Unauthorized"));
    }
  }

  if (payload?.id) {
    req.authUserId = payload.id;
    return next();
  }
  return next(new AppError(401, "Unauthorized"));
};
```

*requireAuthJwtButFlawedSignatureVerification* cũng được tái sử dụng, chỉ khác ở chỗ *verifyUnsignedAccessToken utils*

```jsx
//src\utils\jwt.js
const verifyUnsignedAccessToken = (token) => {
  try {
    const decodedHeader = jwt.decode(token, { complete: true })?.header;
    const secret = decodedHeader?.alg === "none" ? undefined : JWT_SECRET;
    return jwt.verify(token, secret, {
      issuer: refreshTokenOptions.issuer,
      audience: refreshTokenOptions.audience,
      algorithms: [refreshTokenOptions.algorithm, "none"], // HS256 and none
    });
  } catch (error) {
    return null;
  }
};
```

Lỗ hổng cũng xuất phát từ đây *verifyUnsignedAccessToken.* logic của hàm này là nếu như header có alg là “*none*” thì secret sẽ được set là “*undefined”*. Ngoài ra verify nhận algorithms là gồm có “*none*” và algorithm khác được lấy từ biến môi trường. Lỗ hổng này chỉ xảy ra khi *secret* là *undefined* và algorithms nhận thêm *“none”* alg. Lí do là khi truyền một *secret* vào hàm verify nhưng *alg là none*, thư viện sẽ coi đó là một nỗ lực giả mạo và từ chối xác thực. Cụ thể [tại](https://github.com/auth0/node-jsonwebtoken/blob/ed59e76ea37a80f54b833668c02a5271984dcba3/verify.js#L108) 

![jwt2.1.png](jwt2.1.png)

Trong verify có một dòng dùng để kiểm tra khi Signature không có và secret thì có thì sẽ throw error. Nhưng để khai thác none alg thì buộc Signature phải không có. Giờ trường hợp có secret và có signature thì sẽ bypass được dòng đó nhưng ở dòng sau đó ([line](https://github.com/auth0/node-jsonwebtoken/blob/ed59e76ea37a80f54b833668c02a5271984dcba3/verify.js#L165) 165) lại thêm một vấn đề.

![jwt2.2.png](jwt2.2.png)

Tại đây hàm sẽ return lỗi do không valid. Phân tích tiếp jws.verify để hiểu tại sao return false.

![jwt2.3.png](jwt2.3.png)

Xem tiếp [hàm](https://github.com/auth0/node-jws/blob/master/lib/verify-stream.js#L44C1-L44C53) jwsVerify sau khi validate, chuẩn hóa và lấy ra các field như signature, securedInput *(được tạo từ 2 phần là header và payload)* .thì gọi tiếp jwa với algorithm *(ở đây là none)*. Múc đích để lấy ra đối tượng xử lý thuật toán tương ứng.

Module này sử dụng Regex để bóc tách chuỗi thuật toán. Lúc này algo mang giá trị none và bits  là undefined. Tiếp đến nó map kiểu algo là none với các factory tương ứng. Ở đây, đối với việc verify, nó trỏ đến createNoneVerifier, chương trình chính sẽ tiếp tục chạy và đi vào bên trong hàm createNoneVerifier do algo.verify được gọi. Logic sẽ return true nếu signature là rỗng. Mà hiện tại signature đang khác rỗng do trước đó dùng để bypass hàm check trên kia. Hàm return false đồng nghĩa với việc valid mang giá trị false. Đến đây thì chương trình sẽ ném lỗi *“invalid signature”.*

```jsx
valid = jws.verify(jwtString, decodedToken.header.alg, secretOrPublicKey);
...
if (!valid) {
	return done(new JsonWebTokenError('invalid signature'));
}
```

Tránh vỏ dưa, gặp vỏ dừa :))). Đó cũng là lí do mà middlware verifyUnsignedAccessToken mình viết một cách máy móc như vậy và không tự nhiên.

```jsx
// simulate case
const verifyUnsignedAccessToken = (token) => {
  try {
    const decodedHeader = jwt.decode(token, { complete: true })?.header;
    return jwt.verify(token, JWT_SECRET, {
      issuer: refreshTokenOptions.issuer,
      audience: refreshTokenOptions.audience,
      algorithms: decodedHeader?.alg, // HS256 by default is set
    });
  } catch (error) {
    return null;
  }
};
```

Nếu thực tế có đoạn code như trên và set alg là “none” thì vẫn không thể trigger thành công như mình phân tích ở trên. Giờ đi vào khai thác nhé.

![image.png](image-9.png)

Ảnh trên là req, res trước khi attack tại api /profile 

![jwt2.png](jwt2.png)

Đây là sau khi tấn công bằng cách sử dụng json web tokens của burpsuite để modify alg, username và delete signature của jwt. res trả về data của user vul

### Lab: JWT authentication bypass via weak signing key

![image.png](image-10.png)

Tương tự 2 lab trước đó thì cũng sẽ có 2 route */api/v1/jwt/weak-signing-key/login* dùng để verify user credential và tạo jwt access token và */api/v1/jwt/weak-signing-key/profile* dùng để lấy user data cũng là nơi chứa lỗ hổng.

```jsx
//src\routes\v1\jwt.route.js
router.post(
  "/weak-signing-key/login",
  loginRules,
  handleValidation,
  jwtController.loginWeakSigningKey,
);
```

/login để nhận user credential. Cũng giống trước đó chỉ khác ở *loginWeakSigningKey.*

```jsx
//src\controllers\v1\jwt.controller.js
async loginWeakSigningKey(req, res, next) {
  try {
    const user = await jwtService.validateCredentials(req.body); // score up to read
    const token = generateAccessTokenWithWeakSecret(user);
    return successResponse(res, token, "Login successfully");
  } catch (error) {
    next(error);
  }
},
```

generateAccessTokenWithWeakSecret là cái cần quan tâm. Cơ bản code các phần sau sẽ giống nhau thôi chỉ có thêm bớt ở utils nên mình chỉ nói sơ thui.

```jsx
//src\utils\jwt.js
const WEAK_JWT_SECRET = "secret1";
const generateAccessTokenWithWeakSecret = (payload) => {
  return jwt.sign(payload, WEAK_JWT_SECRET, accessTokenOptions);
};
const verifyAccessTokenWithWeakSecret = (token) => {
  try {
    return jwt.verify(token, WEAK_JWT_SECRET, {
      issuer: accessTokenOptions.issuer,
      audience: accessTokenOptions.audience,
      algorithms: [accessTokenOptions.algorithm],
    });
  } catch (error) {
    return null;
  }
};
```

generateAccessTokenWithWeakSecret  return token nhận secrect là từ WEAK_JWT_SECRET có value là secret1 tương tự với verifyAccessTokenWithWeakSecret  dùng để verify, cũng dùng WEAK_JWT_SECRET làm secret. Đây cũng chính là root cause. Nó sử dụng secret dễ đoán thường có thể buteforce khi sử dụng các từ điển có sẵn trên mạng.

```jsx
//src\routes\v1\jwt.route.js
router.get(
  "/weak-signing-key/profile",
  requireAuthJwtButWeakSigningKey,
  profileController.getProfile,
);
```

Route /profile để get user data sử dụng middware requireAuthJwtButWeakSigningKey.

```jsx
//src\middlewares\auth-jwt.middleware.js
const requireAuthJwtButWeakSigningKey = async (req, res, next) => {
  const token = req.get("Authorization")?.split(" ")[1];
  if (!token) {
    return next(new AppError(401, "Unauthorized"));
  }

  const payload = verifyAccessTokenWithWeakSecret(token);

  if (payload?.username) {
    //Use the same username field as portswigger lab
    try {
      const user = await userService.getUserByUsername(payload.username);
      if (!user) {
        return next(new AppError(401, "Unauthorized"));
      }
      req.authUserId = user.id;
      return next();
    } catch {
      return next(new AppError(401, "Unauthorized"));
    }
  }
```

hàm verifyAccessTokenWithWeakSecret ở trên nhó. (”)>

![image.png](image-11.png)

Bài này lúc bật burp suite lên thì nó đã alert lên là sử dụng weak secret rùi. về cơ bản chỉ cần dùng secret xong sign lại jwt sau khi modify là thành công.

![jwt3.png](jwt3.png)

Đây là ảnh khai thác thành công khi sử dụng json web tokens trong burpsuite. Đổi username trong payload thành vul. Sau đó chọn Recalculate Signature và nhập secret1 vào. Extensions sẽ tự động tính toán lại signature và gửi request. Kết quả trong response là dữ liệu của user có username là vul

### Lab: JWT authentication bypass via jwk header injection

![image.png](image-12.png)

Trước khi hiểu jwk là gì thì mình sẽ nói sơ qua về mã hóa bất đối xứng. Mã hóa bất đối xứng sử dụng một cặp khóa gồm khóa riêng tư (private key) và khóa công khai (public key) để thực hiện quá sign và verify token. Khi server tạo JWT, nó sẽ sử dụng private key để sign signature. Sau đó, client sử dụng public key tương ứng để verify signature tương ứng. Điển hình như RS256.

```json
{
  "kty": "RSA",
  "n": "v4ARGCVBUtwQw1Isa-kntUSCkWDuW7EGvu-88BpWkkhiW21ofJHklq-CvFiAckNC5AH8MKHibHpg503KpzNyi48jfy3IFlt7gs2H8r_0JfJDGwmUGcrKvsdiCOxF3e5anjF4LzTSHnHVRlEzLEGo7VUgrnZRLTgJomgKT0268TxLEkvugSabF0Wq3ya4uCzLPK4Ar3iWu28wyS_4RQeaNsZ6xbzXDnDCAMFE44WRgle4__H0nZdxfQGtpqfNbLAqPweU7tOkV_D6I4fPlBPVvxgL4cw6yfD8_mXRyZojolxL8AwoBvmMuWl_GgHcc-Nydn3lWR2yapTUflO-tsl1Iw",
  "e": "AQAB",
  "use": "sig",
  "alg": "RS256",
  "kid": "vullab-rs256-key-1"
}
```

Cấu trúc của nó dạng json obj đại diện cho public key. Có các key fields như kty (key type), kid (key id), use (intended use), alg (algorithm), và các key material fields như n/e cho RSA hoặc x/y cho EC.

Ok trở lại với code lab. Ở đây có 2 route chính */api/v1/jwt/jwk-header-injection/login* và */api/v1/jwt/jwk-header-injection/profile* 

```
//src\routes\v1\jwt.route.js
router.post(
  "/jwk-header-injection/login",
  loginRules,
  handleValidation,
  jwtController.loginJwkHeaderInjection,
);
```

route để nhận user credential và gọi loginJwkHeaderInjection

```jsx
  //src\controllers\v1\jwt.controller.js
  async loginJwkHeaderInjection(req, res, next) {
    try {
      const user = await jwtService.validateCredentials(req.body);
      const token = generateAccessTokenWithRS256Alg(user);
      return successResponse(res, token, "Login successfully");
    } catch (error) {
      next(error);
    }
  },
```

loginJwkHeaderInjection nhận request body và gửi xuống jwtService để validate Credentials. Và tạo jwt access token nếu như không có lỗi gì xảy ra 

```jsx
//src\utils\jwt.js
const generateAccessTokenWithRS256Alg = (payload) => {
  const options = { ...accessTokenOptions, algorithm: "RS256", keyid: JWT_KID };
  return jwt.sign(payload, JWT_PRIVATE_KEY, options);
};
```

generateAccessTokenWithRS256Alg dùng để sign token với JWT_PRIVATE_KEY được lấy từ biến môi trường có format dạng pem. Lí do là vì các thư viện bảo mật thường không hỗ trợ định dạng JSON mà chỉ nhận format pem hay der 

```jsx
//scripts\generate-key.js
const { privateKey, publicKey } = crypto.generateKeyPairSync("rsa", {
  modulusLength: 2048,
  publicKeyEncoding: {
    type: "spki",
    format: "pem",
  },
  privateKeyEncoding: {
    type: "pkcs8",
    format: "pem",
  },
});
```

Bạn có thể tham khảo cách để tạo privateKey, publicKey như code trên nhé.

```jsx
//src\routes\v1\jwt.route.js
router.get(
  "/jwk-header-injection/profile",
  requireAuthJwtButJwkHeaderInjection,
  profileController.getProfile,
);
```

route /profile dùng để lấy user data. và gọi requireAuthJwtButJwkHeaderInjection middlware

```jsx
//src\middlewares\auth-jwt.middleware.js
const requireAuthJwtButJwkHeaderInjection = async (req, res, next) => {
  const token = req.get("Authorization")?.split(" ")[1];
  if (!token) {
    return next(new AppError(401, "Unauthorized"));
  }

  const payload = verifyAccessTokenViaJwk(token);

  if (payload?.username) {
    //Use the same username field as portswigger lab
    try {
      const user = await userService.getUserByUsername(payload.username);
      if (!user) {
        return next(new AppError(401, "Unauthorized"));
      }
      req.authUserId = user.id;
      return next();
    } catch {
      return next(new AppError(401, "Unauthorized"));
    }
  }

  if (payload?.id) {
    req.authUserId = payload.id;
    return next();
  }
  return next(new AppError(401, "Unauthorized"));
};
```

Cũng giống các lab trước chỉ có thay đổi verifyAccessTokenViaJwk utils thôi

```jsx
const verifyAccessTokenViaJwk = (token) => {
  try {
    const decodedHeader = jwt.decode(token, { complete: true })?.header;
    if (decodedHeader?.jwk) {
      const pem = jwkToPem(decodedHeader.jwk);
      return jwt.verify(token, pem, {
        issuer: accessTokenOptions.issuer,
        audience: accessTokenOptions.audience,
        algorithms: ["RS256"],
        keyid: accessTokenOptions.keyid,
      });
    }
    // If user does not send jwk, the public key will be used
    return jwt.verify(token, JWT_PUBLIC_KEY, {
      issuer: accessTokenOptions.issuer,
      audience: accessTokenOptions.audience,
      algorithms: ["RS256"],
      keyid: accessTokenOptions.keyid,
    });
  } catch (error) {
    return null;
  }
};
```

verifyAccessTokenViaJwk support thêm trường jwk từ header và convert sang pem và dùng để verify trực tiếp token. Mình cũng xử lí khi user không gửi jwk lên thì nó sẽ dùng biến môi trường JWT_PUBLIC_KEY thay thế. Và đây cũng chính là lỗ hổng. Nó sử dụng jwk trực tiếp từ header token do user gửi lên. Attacker có thể modify jwk và có thể làm cho token đó hợp lệ. Các solution mình sẽ để ở cuối sau khi hoàn thành hết lab. 

![jwt4.1.png](jwt4.1.png)

Để khai thác lỗ hổng này thì mình sử dụng jwt editor của burpsuite. Sau khi chuyển qua tab jwt editor thì chọn new RSA key và chọn format là jwk và ấn generate và ấn ok. Lúc này jwk để sign sẽ được tạo thành công.

![jwt4.2.png](jwt4.2.png)

Quay lại repeater ấn attack và chọn embedded JWK.

![jwt4.3.png](jwt4.3.png)

Hộp thoại JWT attack sẽ hiện ra lúc này nếu như bạn có nhiều hơn 1 jwk thì nhớ chọn đúng uuid và ấn ok.

![jwt4.4.png](jwt4.4.png)

Lúc này xem lại phần header sẽ thấy jwk đã được thêm vào. Lúc này mình sẽ sửa lại username field trong payload với tên của user có username và vul. Sau đó sign lại với jwk đã được tạo trước đó. Ấn vào nút Sign.

![jwt4.5.png](jwt4.5.png)

 Chọn dont modify header và ấn ok. Nhớ để ý uuid của signing key phải trùng với key mà bạn đã thêm vào phần header trước đó.

![jwt4.6.png](jwt4.6.png)

Sau khi đã modify header, payload và sign lại jwt thì gửi request. Response trả về là user data của user có username là vul. Lúc này có thể thấy là đã khai thác thành công lỗ hổng này.

### Lab: JWT authentication bypass via jku header injection

![image.png](image-13.png)

Lab này sẽ có 2 route chính là */api/v1/jwt/jku-header-injection/login* dùng để verify user credential và tạo jwt access token và */api/v1/jwt/jku-header-injection/profile* dùng để lấy user data

*login* mình sử dụng lại generateAccessTokenWithRS256Alg  utils để tạo jwt access token với alg là RS256 bạn xem lại code ở trước đó nhé.

```jsx
//src\routes\v1\jwt.route.js
router.get(
  "/jku-header-injection/profile",
  requireAuthJwtButJkuHeaderInjection,
  profileController.getProfile,
);
```

route sử dụng middlware requireAuthJwtButJkuHeaderInjection và gửi user data nếu không có lỗi

```jsx
//src\middlewares\auth-jwt.middleware.js
const requireAuthJwtButJkuHeaderInjection = async (req, res, next) => {
  const token = req.get("Authorization")?.split(" ")[1];
  if (!token) {
    return next(new AppError(401, "Unauthorized"));
  }

  const payload = await verifyAccessTokenViaJku(token);

  if (payload?.username) {
    //Use the same username field as portswigger lab
    try {
      const user = await userService.getUserByUsername(payload.username);
      if (!user) {
        return next(new AppError(401, "Unauthorized"));
      }
      req.authUserId = user.id;
      return next();
    } catch {
      return next(new AppError(401, "Unauthorized"));
    }
  }

  if (payload?.id) {
    req.authUserId = payload.id;
    return next();
  }
  return next(new AppError(401, "Unauthorized"));
};
```

requireAuthJwtButJkuHeaderInjection tương tự các lab trước chỉ khác ở verifyAccessTokenViaJku utils

```jsx
const verifyAccessTokenViaJku = async (token) => {
  try {
    //To fit the scenario, instead of using JWT_PUBLIC_KEY directly, we will call api to get jwks
    const decodedHeader = jwt.decode(token, { complete: true })?.header;
    const port = process.env.PORT || 3500;
    const jwksUrl =
      decodedHeader?.jku ??
      `http://localhost:${port}/api/v1/.well-known/jwks.json`;

    if (!decodedHeader.kid) {
      throw new Error("Require kid field in header to fetch Jwks");
    }
    const jwk = await fetchJwkByKid(jwksUrl, decodedHeader.kid);
    if (!jwk) {
      return null;
    }
    const pem = jwkToPem(jwk);
    return jwt.verify(token, pem, {
      issuer: accessTokenOptions.issuer,
      audience: accessTokenOptions.audience,
      algorithms: ["RS256"],
      keyid: accessTokenOptions.keyid,
    });
  } catch (error) {
    return null;
  }
};
```

verifyAccessTokenViaJku  nhận header sau khi decode và kiểm tra trong trường hợp nếu có jku thì gửi fetch tới jku và lấy jwk. Trong trường hợp không gửi thì sẽ dùng hardcode url là *.well-known/jwks.json* để lấy jwk theo kid. 

Tại sao lại có endpoint này thì cần phải hiểu jwks là gì .JSON Web Key Set (JWKS) là một bộ các khóa mã hóa công khai được sử dụng để xác minh tính xác thực và toàn vẹn của các token. Trong khi JWK (JSON Web Key) đại diện cho một khóa mã hóa duy nhất, JWKS là một tập hợp hoặc bộ  các khóa này, thường được cung cấp thông qua một endpoint well-known. 

```jsx
{
  "keys": [
    {
      "kty": "RSA",
      "n": "v4ARGCVBUtwQw1Isa-kntUSCkWDuW7EGvu-88BpWkkhiW21ofJHklq-CvFiAckNC5AH8MKHibHpg503KpzNyi48jfy3IFlt7gs2H8r_0JfJDGwmUGcrKvsdiCOxF3e5anjF4LzTSHnHVRlEzLEGo7VUgrnZRLTgJomgKT0268TxLEkvugSabF0Wq3ya4uCzLPK4Ar3iWu28wyS_4RQeaNsZ6xbzXDnDCAMFE44WRgle4__H0nZdxfQGtpqfNbLAqPweU7tOkV_D6I4fPlBPVvxgL4cw6yfD8_mXRyZojolxL8AwoBvmMuWl_GgHcc-Nydn3lWR2yapTUflO-tsl1Iw",
      "e": "AQAB",
      "use": "sig",
      "alg": "RS256",
      "kid": "vullab-rs256-key-1"
    },
    //more
  ]
}
```

Trên là cấu trúc của jwks và khi client hay service muốn verify bằng public key thì thường sử dụng kid để tìm kiếm jwk tương ứng trong một bộ jwks. kid đại diện và mang tính duy nhất, nó có thể là uuid hay string có thể tự định nghĩa. Đó là lí do vì sao mình sử dụng kid để tìm kiếm trong jwks.json

Do tính chất của lab là sử dụng jwk và được lấy từ endpoint well-knows nên mình đã viết đoạn logic đó. Còn không thì lấy thằng public key từ biến môi trường là xong :))

```jsx
///api/v1/.well-known/jwks.json
router.get("/jwks.json", (req, res, next) => {
  try {
    const jwk = generateJwkFromPem(JWT_PUBLIC_KEY);
    const jwks = {
      keys: [
        {
          ...jwk,
          use: "sig",
          alg: "RS256",
          kid: JWT_KID,
        },
      ],
    };
    res.setHeader("Cache-Control", "public, max-age=3600");
    res.json(jwks);
  } catch (error) {
    next(new AppError(500, error.message));
  }
});

```

endpoint để get jwks nhé. Thì cũng lấy biến môi trường JWT_PUBLIC_KEY dạng pem xong convert sang Jwk obj thôi.

![jwt5.1.png](jwt5.1.png)

Req và res khi gửi tới /jwks.json. 

Như bạn đã thấy thì hàm verifyAccessTokenViaJku nãy cũng là vul chính. Do tin tưởng vào jku trong jwt header do user gửi lên. jku nhằm xác định một URL tham chiếu tới một bộ khóa công khai được đặt ở server. Tới phần khai thác, thao tác dưới đây nhé.

![jwt5.2.png](jwt5.2.png)

Ở đây khi khác thác mình sử dụng jwt editor extension của burpsuite. Khi qua tab jwt editor chọn New RSA Key rồi hộp thoại hiện lên thì chọn jwk format với Id là kid trong jwt header. Ấn generate tiếp và ấn ok

![jwt5.3.png](jwt5.3.png)

Sau khi thành công thì ấn chuột phải vào key vừa tạo và ấn Copy Public Key as JWK. Lưu lại ở đâu đó cho phần sau.

![jwt5.4.png](jwt5.4.png)

Vô webhook và copy đường dẫn chỗ unique URL

![jwt5.5.png](jwt5.5.png)

Ấn edit, hộp thoại thiện ra và đổi Content type là appication/json và Content là jwks theo dưới đây nhé.

```jsx
{
  "keys": [
		// PASTE PUBLIC KEY HERE
  ]
}
```

![jwt5.6.png](jwt5.6.png)

Tiếp đến là thêm field jku và dán url của webhook ở trên vào. Đổi username thành vul. Và ấn sign để sign lại jwt.

![jwt5.7.png](jwt5.7.png)

Hộp thoại hiện ra thì nhớ chọn đúng kid của jwk vừa tạo lúc nãy và chọn dont modify header rồi ấn ok.  

![jwt5.8.png](jwt5.8.png)

Đây là kết quả sau khi gửi modify header, payload và sign lại jwt. Res trả về là data của user có username là vul.

### Lab: JWT authentication bypass via kid header path traversal

![image.png](image-14.png)

Lab này cũng có 2 route chính */api/v1/jwt/kid-header-injection/login* để verify user credential và generate access token với alg là HS256. */api/v1/jwt/kid-header-injection/profile* để lấy user data.

```jsx
//src\routes\v1\jwt.route.js
router.get(
  "/kid-header-injection/profile",
  requireAuthJwtButKidHeaderInjection,
  profileController.getProfile,
);
```

Roure profile để get user data và gọi requireAuthJwtButKidHeaderInjection middlware

```jsx
//src\middlewares\auth-jwt.middleware.js
const requireAuthJwtButKidHeaderInjection = async (req, res, next) => {
  const token = req.get("Authorization")?.split(" ")[1];
  if (!token) {
    return next(new AppError(401, "Unauthorized"));
  }

  const payload = verifyAccessTokenViaKid(token);

  if (payload?.username) {
    //Use the same username field as portswigger lab
    try {
      const user = await userService.getUserByUsername(payload.username);
      if (!user) {
        return next(new AppError(401, "Unauthorized"));
      }
      req.authUserId = user.id;
      return next();
    } catch {
      return next(new AppError(401, "Unauthorized"));
    }
  }

  if (payload?.id) {
    req.authUserId = payload.id;
    return next();
  }
  return next(new AppError(401, "Unauthorized"));
};
```

requireAuthJwtButKidHeaderInjection Tương tự các lab trước chỉ có khác verifyAccessTokenViaKid

```jsx
const verifyAccessTokenViaKid = (token) => {
  try {
    const decodedHeader = jwt.decode(token, { complete: true })?.header;
    if (decodedHeader?.kid) {
      try {
        const pem = fs.readFileSync(path.join(__dirname, decodedHeader.kid), {
          // path traversal
          encoding: "utf-8",
        });
        return jwt.verify(token, pem, {
          issuer: accessTokenOptions.issuer,
          audience: accessTokenOptions.audience,
          algorithms: [accessTokenOptions.algorithm],
        });
      } catch (error) {
        return null;
      }
    }
    return jwt.verify(token, JWT_SECRET, {
      issuer: accessTokenOptions.issuer,
      audience: accessTokenOptions.audience,
      algorithms: [accessTokenOptions.algorithm],
    });
  } catch (error) {
    return null;
  }
};
```

verifyAccessTokenViaKid decode jwt token và lấy kid từ jwt header do user gửi lên và dùng nó làm dường dẫn tới file chứa key. Lỗ hổng cũng xuất phát từ đây. Trong trường hợp không có kid thì vẫn verify bình thường bằng cách sử dụng biến môi trường JWT_SECRET để verify. 

Do rằng mình chạy nodejs trên môi trường window nên cũng không demo bằng cách /dev/null như lab được. Và mình cũng không muốn phức tạp lên tại lười :)). Và không thất thiết là /dev/null chỉ cần attacker biết nội dung của một file bất kì trong hệ thống thì có thể sign lại được jwt. Nên demo thì mình tạo tạm một file k.key nội dung “bellkun”, hacker biết có sự tồn tại của file và nội dung bên trong. Giờ sẽ khai thác nè.

![jwt6.png](jwt6.png)

Mình sử dụng Json web tokens của burpsuite để sign jwt. Bước đầu là đổi kid thành đường dẫn tới file key mà mình control ở đây là ../../k.key và đổi username trong payload thành vul. Sau đó chọn Recacculate Signature và nhập key là bellkun vào ô. Extension sẽ tự động tính toán lại signature. Xong gửi request đi và kết quả là res với data là của user có username là vul.

### Lab: JWT authentication bypass via algorithm confusion

![image.png](image-15.png)

Trước khi vào phân tích thì mình cũng nói sơ quan về mã hóa đối xứng và bất đối xứng. Thì về cơ bản như nãy giờ bạn thấy lỗ hổng trước đó cũng sử dụng 2 loại mã hóa này tương ứng là HS256 cho đối xứng và RS256 cho bất đối xứng. Thì đối với đối xứng thì bạn sử dụng cùng 1 secret key để vừa sign và vừa verify. Còn bất đối xứng thì đòi hỏi sử dụng private key để sign và public key để verify. Nói đến public key thì tên nó cũng nói rõ là public. Và có thể thường public qua endpoint /well-known và do public nên ai cũng có thể có được. Lab này đòi hỏi bạn hiểu qua về 2 loại mã hóa nên mới có thể biết được tại sao dẫn tới vul. Thì về cơ bản ví dụ như trong code sử dụng verify nó nhận không chỉ “RS256” mà đồng thời cũng nhận “HS256” vậy í tưởng là sẽ như thế nào khi sử dụng public key để verify jwt khi sử dụng thuật toán HS256??? View qua code nhé.

Lab mình sử dụng 2 route /api/v1/jwt/algorithm-confusion/login để verify và tạo jwt access token sử dụng thuật toán RS256 sử dụng lại utils generateAccessTokenWithRS256Alg mình đã đề cập ở trên. Và route /api/v1/jwt/algorithm-confusion/profile để get user data.

```jsx
//src\routes\v1\jwt.route.js
router.get(
  "/algorithm-confusion/profile",
  requireAuthJwtButAlgorithmConfusion,
  profileController.getProfile,
);

```

Route sử dụn middlware requireAuthJwtButAlgorithmConfusion

```jsx
//src\middlewares\auth-jwt.middleware.js
const requireAuthJwtButAlgorithmConfusion = async (req, res, next) => {
  const token = req.get("Authorization")?.split(" ")[1];
  if (!token) {
    return next(new AppError(401, "Unauthorized"));
  }

  const payload = await verifyAccessTokenViaAlg(token);

  if (payload?.username) {
    //Use the same username field as portswigger lab
    try {
      const user = await userService.getUserByUsername(payload.username);
      if (!user) {
        return next(new AppError(401, "Unauthorized"));
      }
      req.authUserId = user.id;
      return next();
    } catch {
      return next(new AppError(401, "Unauthorized"));
    }
  }

  if (payload?.id) {
    req.authUserId = payload.id;
    return next();
  }
  return next(new AppError(401, "Unauthorized"));
};
```

middlware này chỉ khác ở utils verifyAccessTokenViaAlg

```jsx
const verifyAccessTokenViaAlg = async (token) => {
  try {
    const decodedHeader = jwt.decode(token, { complete: true })?.header;
    /*
    Bc i using jsonwebtoken in version >= 9, 
    they have added a protection mechanism that 
    you cannot use public key to verify jwt for alg which is HS256
    */
    if (decodedHeader?.alg === "RS256" || decodedHeader?.alg === "HS256") {
      if (!decodedHeader.kid) return null;
      const port = process.env.PORT || 3500;
      const jwksUrl = `http://localhost:${port}/api/v1/.well-known/jwks.json`;
      const jwk = await fetchJwkByKid(jwksUrl, decodedHeader.kid);
      if (!jwk) {
        return null;
      }
      const pem = jwkToPem(jwk);
      // Custom
      if (decodedHeader.alg === "HS256") {
        const [header, payload, signature] = token.split(".");
        const dataToSign = `${header}.${payload}`;

        /*
        Extract raw DER bytes from PEM (strip header/footer, base64-decode).
        This matches how Burp Suite JWT Editor and jwt.io treat the key when
        performing algorithm confusion — they use the raw public key bytes,
        NOT the full PEM string (which includes "-----BEGIN PUBLIC KEY-----",
        newlines, and "-----END PUBLIC KEY-----").
        */
        const pemContent = pem
          .replace("-----BEGIN PUBLIC KEY-----", "")
          .replace("-----END PUBLIC KEY-----", "")
          .replace(/\n/g, "");
        const keyBytes = Buffer.from(pemContent, "base64");

        const expectedSignature = crypto
          .createHmac("sha256", keyBytes)
          .update(dataToSign)
          .digest("base64url");

        if (signature === expectedSignature) {
          return jwt.decode(token);
        }
        return null;
      }
      return jwt.verify(token, pem, {
        issuer: accessTokenOptions.issuer,
        audience: accessTokenOptions.audience,
        algorithms: ["RS256"],
        keyid: accessTokenOptions.keyid,
      });
    }
  } catch (error) {
    return null;
  }
};
```

Code mình viết như vậy là cố tình và không tự nhiên. cũng có lí do của nó

![image.png](image-16.png)

Xem tại [đây](https://github.com/auth0/node-jsonwebtoken/wiki/Migration-Notes:-v8-to-v9). Họ nói rõ rằng trong phiên bản ver 9 đã fix một số security fixes. Trong đó bao gồm cho lỗ hổng Algorithm confusion attacks ở “*Asymmetric keys cannot be used to sign & verify HMAC tokens.”*

Xem kỹ hơn ở phiên bản trước đó nhé. tại [đây](https://github.com/auth0/node-jsonwebtoken/pull/852/files?diff=split&w=0#diff-a32f3d1ddd0e3a886fef0b4523039c3b786a5ac01aea6b13421fa494187762e7L113).

![jwt7.png](jwt7.png)

Trong trường hợp dev không set options.algorithms thì nó sẽ dựa vào secret để xác định alg. Ở đây không có vấn đề gì cả. Nhưng khi dev set algorithms nhận từ jwt alg header do user gửi lên thì sao. Hoặc là như đoạn code dưới 

```jsx
//Case1
jwt.verify(token, pem, {
    algorithms: decodeHeader?.alg
});
//Case2
jwt.verify(token, pem, {
    algorithms: ["HS256", "RS256"]
});
```

Lúc này thì nó sẽ không nhảy vào điều kiện if đó nữa và nhảy thẳng vào jws.verify t

![jwt7.1.png](jwt7.1.png)

Và nếu valid không trả về false thì jwt sẽ được verify thành công.

Xem tiếp [hàm](https://github.com/auth0/node-jws/blob/master/lib/verify-stream.js#L44C1-L44C53) jwsVerify sau khi validate, chuẩn hóa và lấy ra các field như signature, securedInput *(được tạo từ 2 phần là header và payload)* .thì gọi tiếp jwa với algorithm *(ở đây là HS256)*. 
Múc đích để lấy ra đối tượng xử lý thuật toán tương ứng.

![jwt8.png](jwt8.png)

Module này sử dụng Regex để bóc tách chuỗi thuật toán thành kiểu (hs) và số bit (256).

Nó map kiểu hs với các factory tương ứng. Ở đây, đối với việc xác thực (verify), nó trỏ đến createHmacVerifier, module trả về một đối tượng có hai phương thức ở đây là verify đã được thiết lập sẵn số bit tương ứng của thuật toán. Sau khi có đối tượng algo, hàm jwsVerify gọi tiếp algo.verify và lúc này, luồng đi vào bên trong hàm do createHmacVerifier tạo ra

![jwt10.png](jwt10.png)

createHmacVerifier sẽ tính toán Sig dựa trên 2 tham số thing (securedInput tạo từ header + payload) và secrect (secretOrKey) sử dụng hàm crateHmacSigner để tính toán tại Sig dựa trên secret và securedInput. Và so sánh nếu như sig vừa tính toán khác với sig trước đó thì trả về false.
Tóm lại ở đây hàm sẽ kiểm tra tính toàn vẹn của dữ liệu và chọn ra alg tương ứng để tính toán và kiểm tra. Thì có thể nhận thấy rằng không có một cơ chế kiểm tra xem secret có phù hợp với alg tương ứng hay không. Thì bây giờ như 2 case mình trình bày ở trên nếu user gửi lên với alg là HS256 và với secret là public key thì nó vẫn sẽ tính toán và cho ra jwt hợp lệ.

Tuy nhiên nếu như 2 case đó trong trường hợp sử dụng jsonwebtoken thì sẽ không hoạt động

![jwt11.png](jwt11.png)

Lí do là thư viện đã thêm cơ chế và kiểm tra lại loại secret có tương ứng và phù hợp với thuật toán được sử dụng hay không bây giờ nếu sử dụng HS256 và gửi lên public key thì sẽ gây ra lỗi. Và lỗ hổng không hoạt động được. 

Lab sau đó thì về cơ bản cũng na ná lab này thôi. Phần khai thác nào rảnh bổ sung sau, còn nhiều lab khác sợ làm không kịp (”)>

## Solution

Best practice thì cần làm những điều sau:

- Sử dụng và update bản mới nhất của thư viện. Hiểu rõ cách thức hoạt động và những hệ lụy về bảo mật. Các thư viện hiện đại phần nào giúp hạn chế việc triển khai thiếu an toàn một cách vô ý, nhưng không phải lúc nào cũng tuyệt đối.
- Triển khai và xử lí nghiêm ngặt khi sign jwt, đi kèm như bổ sung các field trong jwt như aud, exp,…
- Sử dụng whitelist trong trường hợp có sử dụng jku

```jsx
//src\utils\jwt.js
const verifyAccessTokenViaHS256Alg = (token) => {
  try {
    return jwt.verify(token, JWT_SECRET, {
      issuer: accessTokenOptions.issuer,
      audience: accessTokenOptions.audience,
      algorithms: [accessTokenOptions.algorithm],
    });
  } catch (error) {
    return null;
  }
};
//https://curity.io/resources/learn/jwt-best-practices/
const verifyAccessTokenViaRS256Alg = (token) => {
  try {
    return jwt.verify(token, JWT_PUBLIC_KEY, {
      issuer: accessTokenOptions.issuer,
      audience: accessTokenOptions.audience,
      algorithms: ["RS256"],
      keyid: accessTokenOptions.keyid,
    });
  } catch (error) {
    return null;
  }
};
```

Ở đây dự tính ban đầu mình cũng muốn xử lí phần lấy jwks từ external. Nhưng sử lí ssrf thủ công thì phức tạp. Sử dụng library chuyên vể security thì an toàn hơn. Nhưng do ứng dụng chạy local thôi nên mình cũng không xử lí. Chỉ đọc trực tiếp public key từ biến môi trường. Thế nhé :>>
