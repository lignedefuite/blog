+++
date = '2025-07-01'
draft = false
title = 'SM4端到端加密通信系统 - 核心实现
+++


SM4端到端加密通信系统 - 核心实现
功能：SM4加密引擎 + SM2密钥协商 + 完整的E2EE流程


```python
#!/usr/bin/env python3

# -*- coding: utf-8 -*-



  

import os

import json

import time

import hashlib

import secrets

from typing import Dict, Tuple, Optional, Any

from dataclasses import dataclass, asdict

from cryptography.hazmat.primitives import hashes

from cryptography.hazmat.primitives.kdf.hkdf import HKDF

from cryptography.hazmat.backends import default_backend

  

# 模拟国密算法实现（实际应使用 gmssl 或其他国密库）

class SM4Engine:

"""SM4加密引擎 - 简化实现"""

def __init__(self, mode: str = 'GCM'):

self.mode = mode

self.key_size = 16 # 128 bits

self.block_size = 16

def generate_key(self) -> bytes:

"""生成128位SM4密钥"""

return secrets.token_bytes(self.key_size)

def generate_iv(self) -> bytes:

"""生成初始化向量"""

return secrets.token_bytes(12 if self.mode == 'GCM' else 16)

def encrypt(self, plaintext: bytes, key: bytes, iv: bytes = None) -> Dict[str, bytes]:

"""

SM4加密 (模拟实现)

实际应使用标准SM4算法

"""

if iv is None:

iv = self.generate_iv()

# 简化的加密逻辑（实际应调用标准SM4）

# 这里用AES-GCM模拟SM4-GCM的行为

from cryptography.hazmat.primitives.ciphers.aead import AESGCM

# 警告：这里仅为演示，实际应使用真正的SM4算法

cipher = AESGCM(key)

ciphertext_with_tag = cipher.encrypt(iv, plaintext, None)

# 分离密文和认证标签

ciphertext = ciphertext_with_tag[:-16]

tag = ciphertext_with_tag[-16:]

return {

'ciphertext': ciphertext,

'iv': iv,

'tag': tag

}

def decrypt(self, ciphertext: bytes, key: bytes, iv: bytes, tag: bytes) -> bytes:

"""

SM4解密 (模拟实现)

"""

from cryptography.hazmat.primitives.ciphers.aead import AESGCM

# 重组密文和标签

ciphertext_with_tag = ciphertext + tag

cipher = AESGCM(key)

plaintext = cipher.decrypt(iv, ciphertext_with_tag, None)

return plaintext

  
  

class SM2KeyAgreement:

"""SM2密钥协商 - 简化实现"""

def __init__(self):

self.curve_size = 32 # 256 bits

def generate_keypair(self) -> Tuple[bytes, bytes]:

"""

生成SM2密钥对 (模拟实现)

返回: (private_key, public_key)

"""

# 简化实现：生成随机密钥对

private_key = secrets.token_bytes(self.curve_size)

# 模拟公钥生成（实际应使用椭圆曲线点乘）

public_key = hashlib.sm3(private_key).digest() if hasattr(hashlib, 'sm3') else hashlib.sha256(private_key).digest()

return private_key, public_key

def compute_shared_secret(self, private_key: bytes, peer_public_key: bytes) -> bytes:

"""

计算共享密钥 (模拟ECDH)

"""

# 简化实现：使用哈希模拟ECDH计算

shared_material = private_key + peer_public_key

shared_secret = hashlib.sha256(shared_material).digest()

return shared_secret

def derive_session_key(self, shared_secret: bytes, context: bytes = b'') -> bytes:

"""

派生会话密钥 (使用HKDF模拟SM3-KDF)

"""

hkdf = HKDF(

algorithm=hashes.SHA256(),

length=16, # SM4密钥长度

salt=None,

info=b'SM4-E2EE-Session-Key' + context,

backend=default_backend()

)

return hkdf.derive(shared_secret)

  
  

@dataclass

class E2EEMessage:

"""端到端加密消息结构"""

version: str = "1.0"

message_type: str = "encrypted_message" # key_exchange, encrypted_message

sender_id: str = ""

receiver_id: str = ""

timestamp: float = 0.0

payload: Dict[str, Any] = None

def to_json(self) -> str:

return json.dumps(asdict(self), ensure_ascii=False, indent=2)

@classmethod

def from_json(cls, json_str: str) -> 'E2EEMessage':

data = json.loads(json_str)

return cls(**data)

  
  

class E2EECommunicator:

"""端到端加密通信器"""

def __init__(self, user_id: str):

self.user_id = user_id

self.sm4_engine = SM4Engine()

self.key_agreement = SM2KeyAgreement()

# 生成长期身份密钥对

self.identity_private_key, self.identity_public_key = self.key_agreement.generate_keypair()

# 会话状态

self.sessions = {}

print(f"📱 初始化用户 {user_id}")

print(f"🔑 身份公钥: {self.identity_public_key.hex()[:16]}...")

def initiate_key_exchange(self, peer_id: str) -> E2EEMessage:

"""发起密钥交换"""

# 生成临时密钥对

temp_private, temp_public = self.key_agreement.generate_keypair()

# 保存临时私钥

self.sessions[peer_id] = {

'temp_private_key': temp_private,

'temp_public_key': temp_public,

'status': 'key_exchange_initiated'

}

# 构造密钥交换消息

message = E2EEMessage(

message_type="key_exchange",

sender_id=self.user_id,

receiver_id=peer_id,

timestamp=time.time(),

payload={

'public_key': temp_public.hex(),

'identity_public_key': self.identity_public_key.hex()

}

)

print(f"🤝 {self.user_id} → {peer_id}: 发起密钥交换")

return message

def handle_key_exchange(self, message: E2EEMessage) -> Optional[E2EEMessage]:

"""处理密钥交换消息"""

sender_id = message.sender_id

peer_temp_public = bytes.fromhex(message.payload['public_key'])

# 生成自己的临时密钥对

temp_private, temp_public = self.key_agreement.generate_keypair()

# 计算共享密钥

shared_secret = self.key_agreement.compute_shared_secret(

temp_private, peer_temp_public

)

# 派生会话密钥

session_key = self.key_agreement.derive_session_key(

shared_secret,

(self.user_id + sender_id).encode()

)

# 保存会话信息

self.sessions[sender_id] = {

'temp_private_key': temp_private,

'temp_public_key': temp_public,

'peer_temp_public_key': peer_temp_public,

'session_key': session_key,

'status': 'key_exchange_completed'

}

print(f"🔐 {self.user_id} ↔ {sender_id}: 会话密钥已建立")

print(f" 密钥: {session_key.hex()[:16]}...")

# 回复密钥交换响应

response = E2EEMessage(

message_type="key_exchange_response",

sender_id=self.user_id,

receiver_id=sender_id,

timestamp=time.time(),

payload={

'public_key': temp_public.hex(),

'identity_public_key': self.identity_public_key.hex()

}

)

return response

def complete_key_exchange(self, message: E2EEMessage):

"""完成密钥交换"""

sender_id = message.sender_id

peer_temp_public = bytes.fromhex(message.payload['public_key'])

session = self.sessions[sender_id]

temp_private = session['temp_private_key']

# 计算共享密钥

shared_secret = self.key_agreement.compute_shared_secret(

temp_private, peer_temp_public

)

# 派生会话密钥

session_key = self.key_agreement.derive_session_key(

shared_secret,

(self.user_id + sender_id).encode()

)

# 更新会话状态

self.sessions[sender_id].update({

'peer_temp_public_key': peer_temp_public,

'session_key': session_key,

'status': 'key_exchange_completed'

})

print(f"✅ {self.user_id} ↔ {sender_id}: 密钥交换完成")

print(f" 密钥: {session_key.hex()[:16]}...")

def encrypt_message(self, peer_id: str, plaintext: str) -> E2EEMessage:

"""加密消息"""

if peer_id not in self.sessions:

raise ValueError(f"未找到与 {peer_id} 的会话")

session = self.sessions[peer_id]

if session['status'] != 'key_exchange_completed':

raise ValueError(f"与 {peer_id} 的密钥交换未完成")

session_key = session['session_key']

# 使用SM4加密

result = self.sm4_engine.encrypt(

plaintext.encode('utf-8'),

session_key

)

# 构造加密消息

message = E2EEMessage(

message_type="encrypted_message",

sender_id=self.user_id,

receiver_id=peer_id,

timestamp=time.time(),

payload={

'ciphertext': result['ciphertext'].hex(),

'iv': result['iv'].hex(),

'tag': result['tag'].hex()

}

)

print(f"🔒 {self.user_id} → {peer_id}: 消息已加密")

print(f" 原文: {plaintext}")

print(f" 密文: {result['ciphertext'].hex()[:32]}...")

return message

def decrypt_message(self, message: E2EEMessage) -> str:

"""解密消息"""

sender_id = message.sender_id

if sender_id not in self.sessions:

raise ValueError(f"未找到与 {sender_id} 的会话")

session = self.sessions[sender_id]

if session['status'] != 'key_exchange_completed':

raise ValueError(f"与 {sender_id} 的密钥交换未完成")

session_key = session['session_key']

# 解析密文数据

ciphertext = bytes.fromhex(message.payload['ciphertext'])

iv = bytes.fromhex(message.payload['iv'])

tag = bytes.fromhex(message.payload['tag'])

# 使用SM4解密

plaintext_bytes = self.sm4_engine.decrypt(ciphertext, session_key, iv, tag)

plaintext = plaintext_bytes.decode('utf-8')

print(f"🔓 {sender_id} → {self.user_id}: 消息已解密")

print(f" 密文: {ciphertext.hex()[:32]}...")

print(f" 原文: {plaintext}")

return plaintext

def get_session_info(self, peer_id: str) -> Dict:

"""获取会话信息"""

return self.sessions.get(peer_id, {})

  
  

def demo_e2ee_communication():

"""演示端到端加密通信"""

print("🚀 SM4端到端加密通信演示")

print("=" * 50)

# 创建两个用户

alice = E2EECommunicator("Alice")

bob = E2EECommunicator("Bob")

print("\n" + "=" * 50)

print("📋 阶段1: 密钥协商")

print("=" * 50)

# Alice发起密钥交换

key_exchange_msg = alice.initiate_key_exchange("Bob")

print("\n📤 Alice发送密钥交换请求:")

print(key_exchange_msg.to_json())

# Bob处理密钥交换

key_exchange_response = bob.handle_key_exchange(key_exchange_msg)

print("\n📤 Bob回复密钥交换响应:")

print(key_exchange_response.to_json())

# Alice完成密钥交换

alice.complete_key_exchange(key_exchange_response)

print("\n" + "=" * 50)

print("📋 阶段2: 加密通信")

print("=" * 50)

# Alice发送加密消息

test_messages = [

"Hello Bob! 这是一条测试消息。",

"🔐 端到端加密通信测试成功！",

"密码学真有趣！SM4算法很强大。"

]

for i, msg in enumerate(test_messages, 1):

print(f"\n📨 测试消息 {i}:")

# Alice加密消息

encrypted_msg = alice.encrypt_message("Bob", msg)

# 模拟网络传输（这里直接传递消息对象）

print("\n🌐 网络传输（密文）:")

print(f" {encrypted_msg.payload['ciphertext'][:64]}...")

# Bob解密消息

decrypted_msg = bob.decrypt_message(encrypted_msg)

# 验证消息完整性

assert decrypted_msg == msg, "消息解密失败！"

print("✅ 消息完整性验证通过")

print("\n" + "=" * 50)

print("📊 会话信息统计")

print("=" * 50)

alice_session = alice.get_session_info("Bob")

bob_session = bob.get_session_info("Alice")

print(f"Alice会话状态: {alice_session['status']}")

print(f"Bob会话状态: {bob_session['status']}")

print(f"会话密钥一致性: {alice_session['session_key'] == bob_session['session_key']}")

print("\n🎉 端到端加密通信演示完成！")

  
  

if __name__ == "__main__":

demo_e2ee_communication()
```