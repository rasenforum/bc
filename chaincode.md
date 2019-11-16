# chaincode Sample

## chaincode.go
```
/*
 * SPDX-License-Identifier: Apache-2.0
 */

package main

import (
	"encoding/json"
	"fmt"
	"strings"

	"github.com/hyperledger/fabric/core/chaincode/shim"
	sc "github.com/hyperledger/fabric/protos/peer"
)

// Chaincode is the definition of the chaincode structure.
type Chaincode struct {
}

// TestModel is the definition of asset(model) structure.
type TestModel struct {
	Key   string
	Value string
}

// Init is called when the chaincode is instantiated by the blockchain network.
func (cc *Chaincode) Init(stub shim.ChaincodeStubInterface) sc.Response {
	fcn, params := stub.GetFunctionAndParameters()
	fmt.Println("Init()", fcn, params)
	return shim.Success(nil)
}

// Invoke is called as a result of an application request to run the chaincode.
func (cc *Chaincode) Invoke(stub shim.ChaincodeStubInterface) sc.Response {
	fcn, params := stub.GetFunctionAndParameters()

	if fcn == "putData" {
		return cc.putData(stub, params)
	} else if fcn == "getData" {
		return cc.getData(stub, params)
	}
	return shim.Error("Invalid Chaincode function name")
}

// Define example to call submit transaction
func (cc *Chaincode) putData(stub shim.ChaincodeStubInterface, params []string) sc.Response {
	key := strings.TrimSpace(params[0])

	testModel := TestModel{
		Key:   key,
		Value: params[1],
	}

	valueJSONAsBytes, err := json.Marshal(testModel)

	if err != nil {
		return shim.Error(err.Error())
	}

	err = stub.PutState(key, valueJSONAsBytes)

	if err != nil {
		return shim.Error(err.Error())
	}

	return shim.Success(nil)
}

// Define example to call evaluate transaction
func (cc *Chaincode) getData(stub shim.ChaincodeStubInterface, params []string) sc.Response {
	key := strings.TrimSpace(params[0])

	valueJSONAsBytes, err := stub.GetState(key)

	if err != nil {
		return shim.Error(err.Error())
	}

	return shim.Success(valueJSONAsBytes)
}

```