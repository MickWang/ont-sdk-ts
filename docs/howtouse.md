# Ont SDK 说明文档

## 1 钱包 Wallet

钱包Wallet是一个Jason格式的数据存储文件。在本体Ontology中， Wallet既可存储数字身份，也可以存储数字资产。

### Wallet 数据规范

````
name: string;
    ontid: string;
    createTime: string;
    version: string;
    scrypt: {
        "n": number;
        "r": number;
        "p": number;
    };
    identities: Array<Identity>;
    accounts: Array<Account>;
    extra: null;
````

`name` 是用户为钱包所取的名称。

```ontid``` 是钱包唯一ontid。

```createTime``` 是ISO格式表示的钱包的创建时间，如 : "2018-02-06T03:05:12.360Z"

`version` 目前为固定值1.0，留待未来功能升级使用。

`scrypt` 是加密算法所需的参数，该算法是在钱包加密和解密私钥时使用。

`identities` 是钱包中所有数字身份对象的数组

```accounts``` 是钱包中所有数字资产账户对象的数组

```extra``` 是客户端由开发者用来存储额外数据字段，可以为null。

希望了解更多钱包数据规范请参考[Wallet_File_Specification](https://github.com/ontio/opendoc/blob/master/resources/specifications/Wallet_File_Specification.md).

### 1.1 创建钱包

用户可以从零开始创建自己的钱包。

#### 1）创建一个空的钱包

用户只需要传入钱包名称。

````
import {Wallet} from 'Ont'
var wallet = new Wallet()
wallet.create( name )
````

#### 2) 创建用户的身份并添加到钱包中。

用户需要提供自己的**私钥，身份的名称，密码**来创建身份。你还可以指定创建身份所需的算法对象，该对象应该符合如下结构:

````
{
  algorithm : string,
  parameters : {}
}
````

创建过程默认会提供一个算法对象，如下：

````
{
  algorithm : "ECDSA",
  parameters : {
    "curve": "secp256r1"
}
````

在创建身份的过程中，会生成身份的ONT ID，同时sdk会将该Ont ID发到区块链上注册。

如果注册失败会抛出错误给用户捕获。用户可以重新尝试。

如果注册成功，再将创建好的身份添加到钱包中。默认将身份的Ont ID赋值给钱包。

````
import {Identity} from 'Ont'
var identity = new Identity()
identity.create( privateKey, password, name, algorithmObj )
wallet.ontid = identity.ontid
wallet.addIdentity(identity)
````

#### 3) 创建账户并添加到钱包中

用户需要提供**私钥，密码，账户名称**来创建新的账户。也可以指定创建账户所需的算法对象。创建过程也可以提供默认的算法对象。 同上。

创建好账户后添加到钱包中。

````
import {Account} from 'Ont'
var account = new Account()
account.create( privateKey, password, name )
wallet.addAccount(account)
````

### 1.2 导入新的数字身份到钱包

#### 1）无钱包时导入

这种情况通常发生在用户已经拥有了一个数字身份，但此时客户端应用还没有创建用户钱包文件。此时需要先给用户创建一个空的钱包文件，然后导入用户的身份。

导入数字身份需要用户提供**加密后的私钥，密码，Ont ID**。用户也可以指定导入所需的算法对象。

导入数字身份的过程其实是根据用户提供的信息，创建身份，并加入到钱包的过程。

在导入之前，先要检查用户提供的Ont ID是否存在于本体Ontology。如果不存在，说明身份不合法，此时会抛出错误让用户捕获。

钱包添加身份时会根据身份的Ont ID进行去重判断。 重复的身份不会在钱包中添加多次。

````
import {Wallet} from 'Ont'
import {Identity} from 'Ont'
//1.创建新的钱包
var wallet = new Wallet()
wallet.create( name, password )
//2.创建身份
var identity = Identity.importIdentity( encryptedPrivateKey, password, ontid, algorithmlObj)
//3.添加身份到钱包
wallet.addIdentity(identity)
````

#### 2) 有钱包时导入

过程同上，区别在于用户本地已有钱包。

同样的，创建身份钱先检查用户提供的Ont ID是否存在于链上。

````
import {Identity} from 'Ont'
//1.创建身份
var identity = Identity.importIdentity( encryptedPrivateKey, password, ontid, algorithmlObj)
//2.添加身份到已有钱包
wallet.addIdentity(identity)
````

## 3 Identity

#### Identity 具有以下数据结构

````
{
  ontid : string,
  label : string,
  isDefault : boolean,
  lock : boolean,
  controls : array of controlData,
  extra : null
}
````

```ontid``` 是代表身份的唯一的id

`label` 是用户给身份所取的名称。

`isDefault` 表明身份是用户默认的身份。默认值为false。

`lock` 表明身份是否被用户锁定了。客户端不能更新被锁定的身份信息。默认值为false。

`controls` 是身份的所有控制对象**controlData**的数组。

`extra` 是客户端开发者存储额外信息的字段。可以为null。

#### controlData 具有以下数据结构

````
{
  algorithm: string;
  parameters: {
  	curve: string;
  };
  id: string;
  key: string;
}
````

`algorithm` 是用来加密的算法名称。

`parameters` 是加密算法所需参数。

```curve``` 是椭圆曲线的名称。

`id` 是control的唯一标识。

`key` 是NEP-2格式的私钥。该字段可以为null（对于只读地址或非标准地址时）。

### 2.1 创建身份

在创建身份过程中会将生成的ontid注册到区块链上。注册失败时会抛出相应错误。

````
import {Identity} from 'Ont'
var identity = new Identity()
//@param {string} privateKey 用户私钥
//@param {string} password 密码
//@param {string} label 身份的名称
//@param {object} algorithmObj 可选参数，加密算法对象
try {
  identity.create(privateKey, password, label, algorithmObj)
} catch (error) {
  //注册ontid失败
  console.log(error)
}
````

### 2.2 导入身份

根据用户提供的**加密私钥，密码，ontid** 来创建身份。ontid会赋值到生成的身份上。

````
import {Identity} from 'Ont'
var identity = new Identity()
//@param {string} encryptedPrivateKey 用户加密后的私钥
//@param {string} password 密码
//@param {string} ontid 身份的ontid
//@param {object} algorithmObj 可选参数，加密算法对象
try {
  identity.importIdentity(encryptedPrivateKey, password, ontid, algorithmObj)
} catch (error) {
  //注册ontid失败
  console.log(error)
}
````

## 3 Account

#### Account 具有以下数据结构。

````
{
  address : string,
  label : string,
  isDefault : boolean,
  lock : boolean,
  algorithm : boolean,
  parameters : {
    curve : string
  },
  key : string,
  contract : {}
  extra : null
}
````

```address``` 是base58编码的账户地址。

```label``` 是账户的名称。

`isDefault` 表明账户是否是默认的账户。默认值为false。

`lock` 表明账户是否是被用户锁住的。客户端不能消费掉被锁的账户中的资金。

`algorithm` 是加密算法名称。

`parameters` 是加密算法所需参数。

```curve``` 是椭圆曲线的名称。

`key` 是NEP-2格式的私钥。该字段可以为null（对于只读地址或非标准地址时）。

`contract` 是智能合约对象。该字段可以为null（对于只读的账户地址）。

`extra` 是客户端存储额外信息的字段。该字段可以为null。

#### Contract 具有以下数据结构

````
{
  script : string,
  parameters : [],
  deployed : boolean
}
````

`script` 是智能合约的脚本。当合约已经被部署到区块链上时，该字段可以为null。

`parameters` 是智能合约中函数所需的参数对象，组成的数组。

`deployed` 表明合约是否已被部署到区块链上。默认值为false。

### 3.1 创建账户

````
import {Account} from 'Ont'
var account = new Account()
//@param {string} privateKey 用户的私钥
//@param {string} password 密码
//@param {string} label 账户的名称
//@param {object} algorithmObj 可选参数，加密算法对象
account.create(privateKey, password, label, algorithmObj)
````

### 4 Claim

#### Claim 具有以下数据结构

````
{
  unsignedData : string,
  signedData : string,
  Context : string,
  Id : string,
  Claim : {},
  Metadata : Metadata,
  Signature : Signature
}
````

```unsignedData``` 是未被签名的声明对象的json格式字符串，声明对象包含Context, Id, Claim, Metadata这些字段。

```signedData ``` 是声明对象被签名后的json格式字符串，该json包含声明对象和签名对象。

```Context``` 是声明模板的标识。

```Id``` 是声明对象的标识。

```Claim``` 是声明的内容。

```Metadata``` 是声明对象的元数据。

#### Metadata 具有以下数据结构

````
{
  CreateTime : datetime,
  Issuer : string,
  Subject : string,
  Expires : date,
  Revocation : string,
  Crl : string
}
````

```Createtime``` 是声明的创建时间。

```Issuer``` 是声明的发布者。

```Subject``` 是声明的主语。

```Expires``` 是声明的过期时间。

```Revocation``` 是声明撤销方法。

```Crl``` 是声明撤销列表的链接。

#### Signature 具有以下数据结构

````
{
	Format : string,
    Algorithm : string,
    Value : string
}
````

```Format``` 是签名的格式。

```Algorithm``` 是签名的算法。

```Value``` 是计算后的签名值。

### 4.1 构造声明对象的签名

根据用户输入内容构造声明对象，该声明对象里包含了签名后的数据。

````
import {Claim} from 'Ont'
//@param {string} context 声明模板的标识
//@param {object} claim 用户声明的内容
//@param {object} metadata 声明的元数据
//@param {string} privateKey 用户的私钥
var claim = new Claim(context, claim, metadata, privateKey)
//声明签名后的数据
console.log(claim.signedData)
````





















