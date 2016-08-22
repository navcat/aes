### 加密解密技术方案
1. EncodingAESKey即消息加解密Key，长度固定为43个字符，从a-z,A-Z,0-9共62个字符中选取。由开发者在创建公众号插件时填写，后也可申请修改。
2.AES密钥： AESKey=Base64_Decode(EncodingAESKey + “=”)，EncodingAESKey尾部填充一个字符的“=”, 用Base64_Decode生成32个字节的AESKey；
3.AES采用CBC模式，秘钥长度为32个字节（256位），数据采用PKCS#7填充 ； PKCS#7：K为秘钥字节数（采用32），buf为待加密的内容，N为其字节数。Buf 需要被填充为K的整数倍。在buf的尾部填充(K-N%K)个字节，每个字节的内容 是(K- N%K)。
<tablecellspacing="0" width="700">
    <thead class="thead" style="background-color: rgb(240, 240, 240);">
        <tr class="firstRow">
            <th style="padding: 10px 32px; font-weight: 400; line-height: 20px; border-left-width: 0px; border-left-color: rgb(195, 195, 195); border-bottom-color: rgb(195, 195, 195); text-align: left; min-width: 80px;">尾部填充</th>
            <th class="table_cell no_extra" style="padding: 10px 32px 0px; font-weight: 400; line-height: 20px; border-left-color: rgb(195, 195, 195); border-bottom-color: rgb(195, 195, 195); text-align: left; min-width: 80px;">
                <br>
            </th>
        </tr>
    </thead>
    <tbody class="tbody">
        <tr>
            <td style="padding: 12px 32px; border-top-width: 0px; border-top-color: rgb(230, 230, 230); text-align: left; min-width: 80px;">01</td>
            <td style="padding: 12px 32px; border-top-width: 0px; border-top-color: rgb(230, 230, 230); text-align: left; min-width: 80px;">if ( N%K==(K-1))</td>
        </tr>
        <tr>
            <td style="padding: 12px 32px; border-top-color: rgb(230, 230, 230); text-align: left; min-width: 80px;">0202</td>
            <td style="padding: 12px 32px; border-top-color: rgb(230, 230, 230); text-align: left; min-width: 80px;">if ( N%K==(K-2))</td>
        </tr>
        <tr>
            <td style="padding: 12px 32px; border-top-color: rgb(230, 230, 230); text-align: left; min-width: 80px;">030303</td>
            <td style="padding: 12px 32px; border-top-color: rgb(230, 230, 230); text-align: left; min-width: 80px;">if ( N%K==(K-3))</td>
        </tr>
        <tr>
            <td style="padding: 12px 32px; border-top-color: rgb(230, 230, 230); text-align: left; min-width: 80px;">...</td>
            <td style="padding: 12px 32px; border-top-color: rgb(230, 230, 230); text-align: left; min-width: 80px;">...</td>
        </tr>
        <tr>
            <td style="padding: 12px 32px; border-top-color: rgb(230, 230, 230); text-align: left; min-width: 80px;">KK....KK (K个字节)</td>
            <td style="padding: 12px 32px; border-top-color: rgb(230, 230, 230); text-align: left; min-width: 80px;">if ( N%K==0)</td>
        </tr>
    </tbody>
</table>

具体详见：http://tools.ietf.org/html/rfc2315
4. BASE64采用MIME格式，字符包括大小写字母各26个，加上10个数字，和加号“+”，斜杠“/”，一共64个字符，等号“=”用作后缀填充；
5. 出于安全考虑，开放平台网站提供了修改EncodingAESKey的功能（在EncodingAESKey可能泄漏时进行修改，对应上第三方平台申请时填写的接收消息的加密symmetric_key），所以建议开放平台账号保存当前的和上一次的EncodingAESKey，若当前EncodingAESKey生成的AESKey解密失败，则尝试用上一次的AESKey的解密。回包时，用哪个AESKey解密成功，则用此AESKey加密对应的回包。


### 消息体验证和解密
开发者先验证消息体签名的正确性，验证通过后，再对消息体进行解密。
#### 验证方式：
1. 开发者计算签名，dev_msg_signature=sha1(sort(Token、timestamp、nonce, msg_encrypt))
2. 比较dev_msg_signature和URL上带的msg_signature是否相等，相等则表示验证通过。
#### 解密方式如下：
1. aes_msg=Base64_Decode(msg_encrypt)
2. rand_msg=AES_Decrypt(aes_msg)
3. 验证尾部
4. 去掉rand_msg头部的16个随机字节，4个字节的msg_len,和尾部的