## 背景

### Sample合约

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

contract Sample {
    address public owner;
    constructor() {
        owner = msg.sender;
    }

    function getOwner() external view returns (address) {
        return owner;
    }

    function setOwner(address _owner) public returns (bool) {
        owner = _owner;
        return true;
    }
}
```

使用命令 `solc --ir -o ./ sample.sol` 将sol文件编译成 Sample.yul 文件，部分内容如下：

```yul
/// @use-src 0:"sample.sol"
object "Sample_35" {
    code {
        /// @src 0:58:358  "contract Sample {..."
        mstore(64, memoryguard(128))
        if callvalue() { revert_error_ca66f745a3ce8ff40e2ccaf1ad45db7774001b90d25810abd9040049be7bf4bb() }

        constructor_Sample_35()

        let _1 := allocate_unbounded()
        codecopy(_1, dataoffset("Sample_35_deployed"), datasize("Sample_35_deployed"))

        return(_1, datasize("Sample_35_deployed"))

        function allocate_unbounded() -> memPtr {
            memPtr := mload(64)
        }
        function update_storage_value_offset_0_t_address_to_t_address(slot, value_0) {
            let convertedValue_0 := convert_t_address_to_t_address(value_0)
            sstore(slot, update_byte_slice_20_shift_0(sload(slot), prepare_store_t_address(convertedValue_0)))
        }

        /// @ast-id 12
        /// @src 0:106:155  "constructor() {..."
        function constructor_Sample_35() {

            /// @src 0:106:155  "constructor() {..."

            /// @src 0:138:148  "msg.sender"
            let expr_8 := caller()
            /// @src 0:130:148  "owner = msg.sender"
            update_storage_value_offset_0_t_address_to_t_address(0x00, expr_8)
            let expr_9 := expr_8

        }
        /// @src 0:58:358  "contract Sample {..."

    }
    /// @use-src 0:"sample.sol"
    object "Sample_35_deployed" {
        code {
            /// @src 0:58:358  "contract Sample {..."
            mstore(64, memoryguard(128))
            if iszero(lt(calldatasize(), 4))
            {
                let selector := shift_right_224_unsigned(calldataload(0))
                switch selector
                case 0x13af4035
                {
                    // setOwner(address)
                    external_fun_setOwner_34()
                }
                case 0x893d20e8
                {
                    // getOwner()
                    external_fun_getOwner_20()
                }
                case 0x8da5cb5b
                {
                    // owner()
                    external_fun_owner_3()
                }
                default {}
            }
            revert_error_42b3090547df1d2001c96683413b8cf91c1b902ef5e3cb8d9f6f304cf7446f74()

            function allocate_unbounded() -> memPtr {
                memPtr := mload(64)
            }

            function external_fun_setOwner_34() {

                if callvalue() { revert_error_ca66f745a3ce8ff40e2ccaf1ad45db7774001b90d25810abd9040049be7bf4bb() }
                let param_0 :=  abi_decode_tuple_t_address(4, calldatasize())
                let ret_0 :=  fun_setOwner_34(param_0)
                let memPos := allocate_unbounded()
                let memEnd := abi_encode_tuple_t_bool__to_t_bool__fromStack(memPos , ret_0)
                return(memPos, sub(memEnd, memPos))

            }

            function abi_encode_t_address_to_t_address_fromStack(value, pos) {
                mstore(pos, cleanup_t_address(value))
            }

            function abi_encode_tuple_t_address__to_t_address__fromStack(headStart , value0) -> tail {
                tail := add(headStart, 32)

                abi_encode_t_address_to_t_address_fromStack(value0,  add(headStart, 0))

            }

            function external_fun_getOwner_20() {
                if callvalue() { revert_error_ca66f745a3ce8ff40e2ccaf1ad45db7774001b90d25810abd9040049be7bf4bb() }
                abi_decode_tuple_(4, calldatasize())
                let ret_0 :=  fun_getOwner_20()
                let memPos := allocate_unbounded()
                let memEnd := abi_encode_tuple_t_address__to_t_address__fromStack(memPos , ret_0)
                return(memPos, sub(memEnd, memPos))

            }

            function read_from_storage_split_dynamic_t_address(slot, offset) -> value {
                value := extract_from_storage_value_dynamict_address(sload(slot), offset)

            }

            /// @ast-id 3
            /// @src 0:80:100  "address public owner"
            function getter_fun_owner_3() -> ret {

                let slot := 0
                let offset := 0

                ret := read_from_storage_split_dynamic_t_address(slot, offset)

            }
            /// @src 0:58:358  "contract Sample {..."

            function external_fun_owner_3() {

                if callvalue() { revert_error_ca66f745a3ce8ff40e2ccaf1ad45db7774001b90d25810abd9040049be7bf4bb() }
                abi_decode_tuple_(4, calldatasize())
                let ret_0 :=  getter_fun_owner_3()
                let memPos := allocate_unbounded()
                let memEnd := abi_encode_tuple_t_address__to_t_address__fromStack(memPos , ret_0)
                return(memPos, sub(memEnd, memPos))

            }
            
            function update_byte_slice_20_shift_0(value, toInsert) -> result {
                let mask := 0xffffffffffffffffffffffffffffffffffffffff
                toInsert := shift_left_0(toInsert)
                value := and(value, not(mask))
                result := or(value, and(toInsert, mask))
            }
            
            function update_storage_value_offset_0_t_address_to_t_address(slot, value_0) {
                let convertedValue_0 := convert_t_address_to_t_address(value_0)
                sstore(slot, update_byte_slice_20_shift_0(sload(slot), prepare_store_t_address(convertedValue_0)))
            }

            /// @ast-id 34
            /// @src 0:248:356  "function setOwner(address _owner) public returns (bool) {..."
            function fun_setOwner_34(var__owner_22) -> var__25 {
                /// @src 0:298:302  "bool"
                let zero_t_bool_1 := zero_value_for_split_t_bool()
                var__25 := zero_t_bool_1

                /// @src 0:322:328  "_owner"
                let _2 := var__owner_22
                let expr_28 := _2
                /// @src 0:314:328  "owner = _owner"
                update_storage_value_offset_0_t_address_to_t_address(0x00, expr_28)
                let expr_29 := expr_28
                /// @src 0:345:349  "true"
                let expr_31 := 0x01
                /// @src 0:338:349  "return true"
                var__25 := expr_31
                leave

            }

            /// @ast-id 20
            /// @src 0:161:242  "function getOwner() external view returns (address) {..."
            function fun_getOwner_20() -> var__15 {
                /// @src 0:204:211  "address"
                let zero_t_address_3 := zero_value_for_split_t_address()
                var__15 := zero_t_address_3

                /// @src 0:230:235  "owner"
                let _4 := read_from_storage_split_offset_0_t_address(0x00)
                let expr_17 := _4
                /// @src 0:223:235  "return owner"
                var__15 := expr_17
                leave

            }
            /// @src 0:58:358  "contract Sample {..."

        }

        data ".metadata" hex"a2646970667358221220c3c7bfea64093019fdb0e0aaf2f6130bb86c98f28ea301dcc40713fb9f64853964736f6c634300081c0033"
    }

}
```

## 原理
生成的 Yul 文件是基于以太坊中间表示（Intermediate Representation, IR）的代码，它介于 Solidity 源代码和 EVM 字节码之间。Yul 不是 JavaScript，而是一种低级、高效的语言，用于表示以太坊智能合约逻辑。Yul 的目标是更贴近 EVM 的执行逻辑，同时保留一定的可读性。

### 顶层结构

Yul 文件包含两个主要对象：

1. **`Sample_35`**: 表示主合约的部署代码。
2. **`Sample_35_deployed`**: 表示部署后的运行时代码。

#### 1. `Sample_35`（合约部署阶段）

- **`code`**:
    - 执行合约的部署逻辑，将运行时代码加载到链上。
    - 包含一些辅助函数，如内存分配、错误处理等。

```yul
    object "Sample_35" {
    code {
        mstore(64, memoryguard(128))
```
	
	`mstore(64, memoryguard(128))`: 初始化内存位置，指向位置 `0x40`，这是 Solidity 合约中默认的内存分配起点。

- **检查调用值**：

```yul
        if callvalue() { revert_error_ca66f745a3ce8ff40e2ccaf1ad45db7774001b90d25810abd9040049be7bf4bb() }
```

	执行 `constructor_Sample_35()`，这是对应 Solidity 构造函数 `constructor` 的实现。

- **加载运行时代码**：

```yul
        let _1 := allocate_unbounded()
        codecopy(_1, dataoffset("Sample_35_deployed"), datasize("Sample_35_deployed"))
        return(_1, datasize("Sample_35_deployed"))
```

	将 `Sample_35_deployed` 对象的代码加载到内存中，并将其返回以存储在链上。


#### 2. `Sample_35_deployed`（运行时阶段）

- **代码入口**：

```yul
    object "Sample_35_deployed" {
        code {
            /// @src 0:58:358  "contract Sample {..."
            mstore(64, memoryguard(128))

            if iszero(lt(calldatasize(), 4))
            {
```

	- 如果传入的调用数据长度（`calldatasize()`）大于或等于 4，则认为是函数调用。
	- 读取调用数据的前 4 个字节（`calldataload(0)`），解析为函数选择器（`selector`）。

```yul
                let selector := shift_right_224_unsigned(calldataload(0))
                switch selector

                case 0x13af4035
                {
                    // setOwner(address)

                    external_fun_setOwner_34()
                }

                case 0x893d20e8
                {
                    // getOwner()

                    external_fun_getOwner_20()
                }

                case 0x8da5cb5b
                {
                    // owner()

                    external_fun_owner_3()
                }

                default {}
```

使用 `switch` 根据函数选择器跳转到对应的外部函数实现：

	- `0x13af4035`: 对应 `setOwner(address)`。
	- `0x893d20e8`: 对应 `getOwner()`。
	- `0x8da5cb5b`: 对应 `owner()`。
	- 如果没有匹配的函数选择器，默认执行空操作。


### 构造函数（`constructor_Sample_35`）


```yul
            let expr_8 := caller()
            /// @src 0:130:148  "owner = msg.sender"
            update_storage_value_offset_0_t_address_to_t_address(0x00, expr_8)
            let expr_9 := expr_8
```

- 获取合约部署者地址（`caller()`）并存储到 `owner` 槽位（存储槽位 0）。
- `update_storage_value_offset_0_t_address_to_t_address` 是一个存储更新的通用函数。

### 外部函数实现

#### `setOwner(address)`

```yul
            function external_fun_setOwner_34() {

                if callvalue() { revert_error_ca66f745a3ce8ff40e2ccaf1ad45db7774001b90d25810abd9040049be7bf4bb() }
                
                /// 将地址参数 `_owner` 从调用数据中解码。
                let param_0 :=  abi_decode_tuple_t_address(4, calldatasize())
	            
	            /// 更新存储
                let ret_0 :=  fun_setOwner_34(param_0)
                let memPos := allocate_unbounded()
                let memEnd := abi_encode_tuple_t_bool__to_t_bool__fromStack(memPos , ret_0)
                return(memPos, sub(memEnd, memPos))

            }
```

更新存储：
```yul
            function fun_setOwner_34(var__owner_22) -> var__25 {
                /// @src 0:298:302  "bool"
                let zero_t_bool_1 := zero_value_for_split_t_bool()
                var__25 := zero_t_bool_1

                /// @src 0:322:328  "_owner"
                let _2 := var__owner_22
                let expr_28 := _2
                /// @src 0:314:328  "owner = _owner"
                /// 将 `_owner` 更新到 `owner` 的存储槽位（0）。
                update_storage_value_offset_0_t_address_to_t_address(0x00, expr_28)
                let expr_29 := expr_28
                /// @src 0:345:349  "true"
                let expr_31 := 0x01
                /// @src 0:338:349  "return true"
                var__25 := expr_31
                leave

            }
```

	- 返回布尔值 `true`，表示操作成功。

#### `getOwner()`


```yul
            function external_fun_getOwner_20() {

                if callvalue() { revert_error_ca66f745a3ce8ff40e2ccaf1ad45db7774001b90d25810abd9040049be7bf4bb() }
                abi_decode_tuple_(4, calldatasize())

				/// 从存储槽位读取 `owner`
                let ret_0 :=  fun_getOwner_20()
                
                let memPos := allocate_unbounded()
                let memEnd := abi_encode_tuple_t_address__to_t_address__fromStack(memPos , ret_0)
                /// 将结果编码为 ABI 格式并返回。
                return(memPos, sub(memEnd, memPos))

            }
```


```yul
            function fun_getOwner_20() -> var__15 {
                /// @src 0:204:211  "address"
                let zero_t_address_3 := zero_value_for_split_t_address()
                var__15 := zero_t_address_3

                /// @src 0:230:235  "owner"
                /// 读取存储槽位 0 的值，即当前合约的 `owner` 地址。
                let _4 := read_from_storage_split_offset_0_t_address(0x00)
                let expr_17 := _4
                /// @src 0:223:235  "return owner"
                var__15 := expr_17
                leave

            }
```

### 辅助函数

- **`allocate_unbounded`**: 分配内存。
- **`abi_decode` 和 `abi_encode`**: 解码和编码 ABI 数据。
- **`cleanup`**: 用于数据清理，确保值符合地址或布尔格式。
- **`shift`**: 位操作，用于提取或存储特定位数据。

### 总结

- 该 Yul 文件表示了 `Sample` 合约的部署和运行时逻辑，分为部署代码和运行时代码。
- 核心逻辑如存储更新、函数选择、参数解码/编码都通过低级 Yul 操作实现。
- Yul 提供了更高效、更贴近 EVM 的执行语义，同时保留了逻辑清晰度。