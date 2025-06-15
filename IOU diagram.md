flowchart TD
    %% Deployment and Owner Initialization
    A[Contract Deployment] --> B[owner = msg.sender]
    B --> C[registeredFriends[msg.sender] = true]
    C --> D[friendList.push(msg.sender)]

    %% Registration unlocks all user actions
    E[addFriend(_friend)] --> F{Valid & not already registered?}
    F -- Yes --> G[registeredFriends[_friend] = true]
    G --> H[friendList.push(_friend)]
    G --> I[User can now deposit, record IOU, pay, transfer, withdraw]
    F -- No --> J[Fail]

    %% Deposit increases balance
    K[Registered friend calls depositIntoWallet()]
    K --> L{msg.value > 0?}
    L -- Yes --> M[balances[msg.sender] += msg.value]
    M --> N[balance[msg.sender] updated]
    L -- No --> O[Fail]

    %% IOU creation depends on registration
    P[Registered friend calls recordDebt(_debtor, _amount)]
    P --> Q{_debtor valid & registered? _amount > 0?}
    Q -- Yes --> R[debts[_debtor][msg.sender] += _amount]
    Q -- No --> S[Fail]

    %% Paying IOU updates balances and debts
    T[Debtor calls payFromWallet(_creditor, _amount)]
    T --> U{All checks: registered, debt, balance}
    U -- Yes --> V[balances[msg.sender] -= _amount]
    V --> W[balances[_creditor] += _amount]
    W --> X[debts[msg.sender][_creditor] -= _amount]
    U -- No --> Y[Fail]

    %% Direct transfer requires registration and balance
    Z[transferEther/_ViaCall(_to, _amount)]
    Z --> AA{Checks: registered, balance}
    AA -- Yes --> AB[balances[msg.sender] -= _amount]
    AB --> AC[Send Ether to _to]
    AC --> AD[balances[_to] += _amount]
    AA -- No --> AE[Fail]

    %% Withdrawal
    AF[withdraw(_amount)]
    AF --> AG{Registered & balance?}
    AG -- Yes --> AH[balances[msg.sender] -= _amount]
    AH --> AI[Send Ether to msg.sender]
    AG -- No --> AJ[Fail]

    %% Query
    AK[checkBalance()] --> AL[Returns balances[msg.sender]]

    %% Highlight core links
    subgraph StateVariables [Shared State]
        N
        R
        V
        W
        X
        AB
        AD
        AH
        AL
    end

    %% Connect all state updates to StateVariables
    M --> StateVariables
    R --> StateVariables
    V --> StateVariables
    W --> StateVariables
    X --> StateVariables
    AB --> StateVariables
    AD --> StateVariables
    AH --> StateVariables
    AL --> StateVariables
