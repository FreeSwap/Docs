# <center> FreeSwap 免费交易协议 </center>

[TOC]

## 摘要
FreeSwap 交易协议是一种不收取交易手续费的去中心化交易协议。FreeSwap 交易协议以 “恒定资产乘积” 公式为基础，为每对数字资产创建两个单向交易的子资金池，随着交易发生，当两个子资金池内的资产价格偏差达到一定幅度时，子资金池之间相互进行套利操作。套利操作可以恢复子资金池资产价格的一致性，同时为流动性资金提供者提供收益。本文主要描述 FreeSwap 交易协议的套利机制及其基本规则，阐述套利机制的双赢特性，并通过理论模型对套利机制的收益幅度进行量化评估。计算表明，在子资金池价差为 1 % 的情况下进行套利，FreeSwap 交易协议可以实现相当于 2.488 ‰ 交易手续费的套利收益。

## 0 引言

2020年，"恒定资产乘积" 公式 $^{[1]}$ 应用到去中心化交易所取得了巨大成功。"恒定资产乘积" 公式异常简洁，但却能够解决链上资产的自动化定价问题，UniSwap $^{[2]\ [3]}$ 等基于 "恒定资产乘积" 公式的去中心化交易所可以提供非常好的流动性和交易深度，吸引了大量的加密货币交易用户。

但是去中心化交易除了需要支付区块链网络的Gas费外，还需要支付一定比例的交易手续费，相对于中心化交易所，用户需要支付较高的交易费用。FreeSwap 交易协议目的是设计一种完全免费的不需要支付交易手续费的交易协议，通过降低交易手续费，吸引费更多的去中心化交易用户。

为解决 "抢跑攻击" 问题，Vitalik Buterin 曾经提出设置两个单向交易池的建议 $^{[4]}$，FreeSwap 协议具体细化了这一建议。由于两个交易池只能进行单向的资产兑换，这必然会导致交易池之间资产价格产生反向偏离，这种反向偏离既可以解决矿工的 "抢跑攻击” 问题，又可以给流动性提供者提供额外的盈利途径。FreeSwap 交易协议充分利用两个子资金池资产价格反向偏离产生的套利机会，给流动性资金提供者提供收益，进而为去中心化交易用户提供免费交易。

## 1 交易对基础

### 1.1 交易对定义

交易对包括两种不同的可以相互交换的数字资产，这里分别用 $Token A$ 和 $Token B$ 表示。在创建交易对时，用户需要按照实际市场价格存入相应数量的价值相等的 $Token A$ 和 $Token B$ 。如果创建交易对的价格偏离市场价格，交易对创建者会被套利，从而蒙受损失。

假设 $P_{A}$ 、$P_{B}$ 分别为 $Token A$ 、$Token B$ 以任一法币计价的价格，定义 $P_{A\rightarrow B}$ 为 $Token A$ 兑换为 $Token B$ 的价格，即与 $1\ Token A$ 等值的 $Token B$ 的数量：

$$P_{A \rightarrow B} = \frac{P_A}{P_B} \tag{1.1.1}$$

同样，定义 $P_{B \rightarrow A}$ ，即 $Token B$ 兑换为 $Token A$ 的价格：

$$P_{B \rightarrow A} = \frac{P_B}{P_A} \tag{1.1.2}$$

理论上，不考虑市场交易成本，则存在下列关系：

$$P_{A \rightarrow B} * P_{B \rightarrow A} = 1 \tag{1.1.3}$$

假设 $N_A$ 、$N_B$ 分别为交易对内部任一时刻两种数字资产 $Token A$、 $Token B$ 的数量，该交易对可表示为：

$$(N_A \ | \ N_B) \tag{1.1.4}$$

交易对总是认为两种内部资产具有相等的市场价值，即存在下列相互等价的关系：

$$N_A * P_{A} = N_B * P_{B} \tag{1.1.5}$$

$$N_A = N_B * P_{B \rightarrow A} \tag{1.1.6}$$ 

$$N_B = N_A * P_{A \rightarrow B} \tag{1.1.7}$$

$$P_{A \rightarrow B} = \frac{N_B}{N_A} \tag{1.1.8}$$

$$P_{B \rightarrow A} = \frac{N_A}{N_B} \tag{1.1.9}$$

### 1.2 恒定资产乘积

在交易对中进行资产兑换时，由于无法取得外部 $Token A$ 和 $Token B$ 的市场价格，需要设计一个兑换机制，确定 $Token A$ 和 $Token B$ 兑换比例关系。自动做市商 (Automated Market Maker，AMM) 类型的去中心化交易所 (DEX)，采用 “资产恒定乘积” 公式 $^{[1]}$ 确定兑换前后交易对中资产数量的变化，也即用户兑入、兑出资产的数量；

对于交易对 $(N_A\ | \ N_B )$ ，"恒定资产乘积" 公式可表示为：

$$N_A * N_B=K \tag{1.2.1}$$

其中，$K$ 值只在用户向交易对中存入流动性资产，或者从交易对中取出流动性资产时才会发生变化，在用户兑换资产的交易过程中保持恒定不变。

假设有一笔兑换交易，用户用 $Token A$ 兑换 $Token B$ ，兑入的 $Token A$ 的数量为 $\mathrm{\Delta}_{A}$ ，兑出的 $Token B$ 的数量为 $\mathrm{\Delta}_{B}$ 。（为表述方便，这里的 “兑入”、“兑出” 是相对交易对而言的，如果相对用户而言，“兑入”、“兑出” 关系则完全相反）。依据 "恒定资产乘积" 公式，有：

$$ ( N_A + \mathrm{\Delta}_A ) * (N_B - \mathrm{\Delta}_B ) = ( N_A * N_B )  \tag{1.2.2}$$

$$\mathrm{\Delta}_B = \frac{\mathrm{\Delta}_A}{N_A + \mathrm{\Delta}_A} * N_B \tag{1.2.3}$$

可以看到，兑换交易发生前、兑换交易发生后、以及兑换交易实际发生的价格是3个不同的价格。

兑换交易发生前，交易对内的资产价格（表示为 $P_{A \rightarrow B}^{0}$）为：

$$P_{A \rightarrow B}^{0} = \frac{N_B}{N_A} \tag{1.2.4}$$

兑换交易实际发生的资产价格（表示为 $P_{A \rightarrow B}^{1}$）为：

$$P_{A \rightarrow B}^{1} = \frac{\mathrm{\Delta}_B}{\mathrm{\Delta}_A} = \frac{N_B}{N_A + \mathrm{\Delta}_A} \tag{1.2.5}$$

兑换交易发生后，交易对内的资产价格（表示为 $P_{A \rightarrow B}^{2}$）为：

$$P_{A \rightarrow B}^{2} = \frac{N_B - \mathrm{\Delta}_B}{N_A +  \mathrm{\Delta}_A} \tag{1.2.6}$$

显而易见：

$$P_{A \rightarrow B}^{0} > P_{A \rightarrow B}^{1} > P_{A \rightarrow B}^{2} \tag{1.2.7}$$

即兑入资产 $Token A$ 相对于兑出资产 $Token B$ 的价格，在兑换交易发生后，发生了滑动。兑换交易发生前交易对内 $Token A$ 的价格，高于用户兑换交易实际发生的价格，而用户兑换交易实际发生的价格，又高于兑换完成后交易对内 $Token A$ 的价格。即兑换交易完成后，交易对内 $Token A$ 相对于 $Token B$ 的价格发生了下跌，相应地，$Token B$ 的相对价格产生了上涨。

### 1.3 交易套利

兑换交易是在交易区块确认的瞬间发生的，兑换交易发生前后，$Token A$ 和 $Token B$ 的外部市场价格不会发生变化。由于 "恒定资产乘积" 约束，导致交易价格产生滑动，会使用户蒙受一定的兑换损失，以 $Token B$ 计价的兑换损失计算如下：

$$\begin{aligned} Lost_{A \rightarrow B} &= \mathrm{\Delta}_A * \frac{N_B}{N_A} - \mathrm{\Delta}_A * \frac{N_B}{N_A + \mathrm{\Delta}_A} \\[3mm]
&= \mathrm{\Delta}_A * \frac{N_B}{N_A} \Large{/} \normalsize ( 1 + \frac{N_A} {\mathrm{\Delta}_{A}} ) \end{aligned} \tag{1.3.1}$$

可见用户兑入Token A的比列 $(\mathrm{\Delta}_{A} / N_A)$  越大，蒙受的兑换损失也会越大。

该交易完成后，如果另一用户进行逆向兑换，不考虑交易费用，根据 "恒定资产乘积" 公式，他只要兑入数量为 $\mathrm{\Delta}_B$ 的 $Token B$ ，即可获得 $\mathrm{\Delta}_A$ 的 $Token A$ ，交易完成后，交易对内资产价格恢复为初始价格 $P_{A \rightarrow B} = N_B/N_A$ 。即该用户以较高的 $Token B$ 价格对较低的 $Token A$ 价格完成交易，实现了对前一用户兑换损失的套利，套利金额等于前一用户的兑换损失。

### 1.4 无常损失

流动性资金提供者在提供流动性时，需要向交易对中注入两种总价值相等的相应数量的代币，随着交易的发生，用户权益对应的代币数量会发生变动。假设用户初始投入 $Token A$ 的数量为 $X_{A}$ ，当用户退出流动性时，如果 $Token A$ 的市场价格相对于 $Token B$ 发生了上涨，则用户得到的 $Token A$ 的数量 ${X'}_{A}$ 可能会小于 $X_{A}$，如果提供流动性获得的交易手续费等收入不足以弥补 $Token A$ 数量减少造成的损失，则流动性提供者相对于一直持有 $Token A$ 、$Token B$ 而不是提供流动性服务，则会蒙受损失。实际上，无论 $Token A$、$Token B$的价格上涨还是下跌，只要他们的价格比相对于用户提供流动性时发生了偏离，用户都会蒙受损失。该损失大小完全由市场价格变化决定，用户无法控制，也被称为 "无常损失" $^{[5]}$。

下面以 $Token B$ 作为价格基准，在不考虑交易手续费收入的情况下，对无常损失进行定量分析。假设初期用户投入交易对的代币数量为 $(X_{A}|Y_{B})$ ，$Token A$ 相对于 $Token B$ 的价格为 $P_1$，期末用户撤出交易对的代币数量为 $(X'_A | Y'_B)$，$Token A$ 相对于 $Token B$ 的价格为 $P_2$，如果用户没有为交易池提供流动性，而是简单持有 $( X_{A}|Y_{B} )$ 代币，期末价值为：

$$V_1=X_A*P_2+Y_B \tag{1.4.1}$$

由于用户为交易池提供流动性，实际拥有 $(X'_{A}|Y'_{B})$ 代币的价值为：

$$V_2=X'_A*P_2+Y'_B \tag{1.4.2}$$

考虑到：

$$\left\{ \begin{matrix} \begin{aligned}
X_A * Y_B &= X'_A * Y'_B \\
P_1 &= Y_B / X_A \\
P_2 &= Y'_B / X'_A \\
\end{aligned} \end{matrix} \right. \tag{1.4.3}$$

则有 $^{[6]}$：

$$\begin{aligned} \frac{V_2}{V_1} &= \frac{\ 2\sqrt{P_1*P_2}\ }{P_1+P_2} \\[3mm]
&= \sqrt{\frac{4P_{1}P_{2}}{( P_{1} - P_{2} )^{2} + 4P_{1}P_{2}}} \leq 1 \end{aligned} \tag{1.4.4}$$

可见只要 $P_1 \neq P_2$，$V_2$ 就总是小于 $V_1$，即用户以 $Token B$ 计价的总资产价值相对于简单持有两种代币，一定是减少的。同理用户以 $Token A$ 计价的总资产价值相对于简单持有两种代币也一定是减少的，所以所谓的无常损失（Impermanent Loss）实际上是一定会发生的永久损失（Permanent Loss），只是该损失金额会随着价格的变动而变动。

## 2. FreeSwap 交易协议

### 2.1 FreetSwap 交易协议目标

FreeSwap 交易协议目标实现一个完全没有交易手续费的去中心化交易所。"恒定资产乘积" 交易机制不可避免地产生交易价格滑动，FreeSwap 协议通过从滑动价格中自动套利，为流动性提供者提供收益来源。

### 2.2 FreeSwap 交易对设置

FreeSwap 交易协议为代币交易对 $(Token_A|Token_B)$ 设置两个独立的单向子交易池，表示为：

$$( N_{AA} | N_B)\ || \ (N_A | N_{BB} ) \tag{2.2.1} $$

其中, 子交易对 $(N_{AA}|N_B )$ ( 简称 $A$ 子交易对 、$A$ 池 ) 可以单向兑入 $Token A$ 、兑出 $Token B$，$N_{AA}$ 为 $A$ 池内 $Token A$ 的数量，$N_B$ 为 $A$ 池内 $Token B$ 的数量。

子交易对 $(N_A|N_{BB} )$ ( 简称 $B$ 子交易对 、$B$ 池 ) 可以单向兑入 $Token B$ 、兑出 $Token A$，$N_{BB}$ 为 $B$ 池内 $Token B$ 的数量，$N_A$ 为 $B$ 池内 $Token A$ 的数量。

流动性提供者向交易池注入资金时，可以指定所要注入的子交易池，也可以设定一定的比例，将资金同时注入两个子交易池。

### 2.3 FreetSwap 兑换交易

用户跟交易池进行 $Token A$ 与 $Token B$ 的兑换交易时，会有2种相反的交易操作，一是兑入 $Token A$ ，兑出 $Token B$ ；一是兑入 $Token B$ ，兑出 $Token A$ 。这里 “兑入” 和 “兑出” 是相对交易池而言的。

如果用户兑入 $Token A$ 、兑出 $Token B$ ，FreeSwap 会将该交易交给 $A$ 池, 即 $(N_{AA}|N_B )$ 子交易对完成。$A$ 池总是兑入 $Token A$ ，兑出 $Token B$ ，所以 $N_{AA}$ 的数量一直会上升，$N_B$ 的数量一直会下降，这会导致 $A$ 池内以 $Token B$ 计价的 $Token A$ 的价格会单调下降，导致 $Token A$ 的价格偏离实际市场价格，这种价格偏离可以为 FreeSwap 交易对提供套利空间。

同样，如果用户兑入 $Token B$ ，兑出 $Token A$ ，会由 $B$ 池来完成交易。$B$ 池总是兑入 $Token B$ ，兑出 $Token A$ ，所以 $N_{BB}$ 的数量会一直上升，$N_A$ 的数量会一直下降，这同样会导致 $B$ 池内以 $Token A$ 计价的 $Token B$ 的价格单调下降，最终偏离实际市场价格。

子交易对 $(N_{AA} | N_B )$，$(N_A | N_{BB} )$ 是两个单向的方向相反的交易对，它们总是固定兑入一种代币，兑出另一种代币。随着交易的发生，两种代币的价格在两个子交易对内总是以相反的方向滑动，积累一定交易量后，同一代币在两个子交易对内的价差就会超过一定的幅度。价差过大会阻止用户继续参与兑换交易，这时需要在两个子交易对内进行两种资产的资金调配，让代币的兑换价差得到恢复。在两个子交易对内进行资金调配的过程实际上是一个交易套利过程。交易用户在两个单向交易池内进行代币交易，造成代币价格单向滑动，用户会因此蒙受价格滑动造成的交易损失。通过两个子交易池相互进行资产调配，既可以对价格滑动进行修复，又可以实现对用户交易滑动损失的套利，为交易池流动性提供者带来套利收益。

### 2.4 FreeSwap 交易池套利

如上所述，FreeSwap 设置两个独立的子交易对：

$$( N_{AA} | N_B )\ || \ ( N_A | N_{BB} ) \tag{2.4.1}$$

$Token A$ 、$Token B$ 在两个子交易对内的价格随着交易发生会产生偏离，当价格偏离较大时，需要对两个子交易对进行资产调配，恢复资产价格。两个子交易对之间的资产调配既可以恢复双方的资产价格，也可以实现相互之间的套利，下面进行分析。

套利前，$Token A$ 在两个交易对内的价格为：

$$P_{AA \rightarrow B} = \frac{N_B}{N_{AA}} \tag{2.4.2}$$

$$P_{A \rightarrow {BB}} = \frac{N_{BB}}{N_A} \tag{2.4.3}$$

其偏离比例为：

$$R_{PA} = \frac{P_{A \rightarrow BB}}{P_{AA \rightarrow B}} = \frac{N_{AA} * N_{BB}}{N_B \ * \ N_A} \tag{2.4.4}$$

同理，$Token B$ 在两个交易对内的价格为：

$$P_{BB \rightarrow A} = \frac{N_A}{N_{BB}} \tag{2.4.5}$$

$$P_{B \rightarrow {AA}} = \frac{N_{AA}}{N_B} \tag{2.4.6}$$

其价格偏离比例为：

$$R_{PB} = \frac{P_{B \rightarrow AA}}{P_{BB \rightarrow A}} = \frac{N_{AA} * N_{BB}}{N_B * N_A} \tag{2.4.7}$$

可见，两个子交易对内 $Token A$，$Token B$ 的价格比相同，即：

$$R_{PA} = R_{PB} = R_P = \frac{N_{AA} * N_{BB}}{N_B * N_A} \tag{2.4.8}$$

FreeSwap 协议设定当两个子交易对的资产价格偏离，即 $R_{P}$ 的值大于一定门限值 $\gamma$，如101% 时 ，自动启动内部套利机制，恢复价格。

套利操作就是利用一个子交易对内数量较多的资产，等价值兑换另一个子交易对内数量较多的另一个资产，即 $( N_{AA} | N_B )$ 子交易对利用 $Token A$ 换取 $( N_A | N_{BB} )$ 子交易对等价值的 $Token B$ 。换个角度看，等价于 $( N_A | N_{BB} )$ 子交易对利用 $Token B$ 换取 $( N_{AA} | N_B )$ 子交易对等价值的$Token A$ 。

为保证兑换公平，兑换价格定为两个子交易对所有资产的平均价格，即：

$$P_{A \rightarrow B}^e = \frac{N_B + N_{BB}}{N_A + N_{AA}} \tag{2.4.9}$$

$$P_{B \rightarrow A}^e = \ \frac{N_A + N_{AA}}{N_B + N_{BB}} \tag{2.4.10}$$

$$P_{A \rightarrow B}^e * P_{B \rightarrow A}^e = \ 1 \tag{2.4.11}$$

套利操作完成后，两个子交易对可以表示为：

$$(N_{AA} - L_A \ | \ N_B + L_B  )\ ||\ ( N_A + L_A \ | \ N_{BB} - L_B ) \tag{2.4.12}$$

即在交易对套利操作中，$A$ 池利用数量为 $L_A$ 的 $Token A$ 换取数量为 $L_B$ 的 $Token B$ ，考虑交换价值相等，则存在下列关系：

$$L_{B} = L_{A}*P_{A \rightarrow B}^e \tag{2.4.13}$$

套利操作完成后，$A$ 池的 $Token A$ 价格（以 $Token B$ 计价）将会上涨，但基于合理性考虑，不应该超过整体交易对的平均价格 $P_{A \rightarrow B}^e$ ，同样 $B$ 池的 $Token B$ 价格（以 $Token A$ 计价）也会上涨，但也不应该超过整体交易对的平均价 $P_{B \rightarrow A}^e$ ，即存在下列关系：

$$\frac{N_B + L_B}{N_{AA} - L_A} \leq P_{A \rightarrow B}^e \tag{2.4.14} $$

$$\frac{N_A \ + \ L_A}{N_{BB} \ - \ L_B} \normalsize \leq P_{B \rightarrow A}^e \tag{2.4.15}$$

结合关系式 $(2.4.13)-(2.4.15)$, 可得出下列关系: 

$$L_A \leq \frac{N_{AA}*N_{BB} - N_A*N_B}{2*(N_B + N_{BB})}  \tag{2.4.17} $$

$$L_B \leq \frac{N_{AA}*N_{BB} - N_A*N_B}{2*(N_A + N_{AA})}  \tag{2.4.18} $$

为减少套利操作频次，子资金池之间的套利操作应当尽量多地交换两种资产以最大幅度地恢复资产价格偏离，不难发现，当 $L_A$, $L_B$ 取值上述关系式中的最大值时，两个子交易对的资产价格恰好同时恢复到这个交易池的资产平均价格 $P_{A \rightarrow B}^e$，这也是双方交换资产的价格。

总体而言，FreeSwap 套利操作可以表示如下：

$$\left\{ \begin{aligned} 
\ \ &\frac{N_{AA} * N_{BB}}{N_A * N_B} \ \geq \ \gamma \\[3mm]
P_{A \rightarrow B}^e \ = \ \ &\frac{N_B + N_{BB}}{N_A + N_{AA}} \\[3mm]
L_A^e \ = \ \ &\frac{N_{AA}*N_{BB} - N_A*N_B}{2*(N_B + N_{BB})}  \\[3mm]
L_B^e \ = \ \ &\frac{N_{AA}*N_{BB} - N_A*N_B}{2*(N_A + N_{AA})}  \\[3mm]
\end{aligned} \right. \tag{2.4.19} $$ 

其中, $\gamma$ 是触发套利操作的条件，$\gamma$ 表示当两个子交易池的价格偏离大于等于 $\gamma-1$ 时触发套利操作。$L_A^e$、$L_B^e$ 是套利操作时两个子资金池相互交换资产的数量，套利操作按照 $P_{A \rightarrow B}^e$ 的价格相互交换 $L_A^e$、$L_B^e$数量的 $TokenA$，$Token B$，套利操作完成后，两个子资金池的资产价格完全相同，均为 $P_{A \rightarrow B}^e$。

## 3 FreeSwap 套利分析

### 3.1 FreeSwap 套利分析

从 $A$ 池、$B$ 池的流动性资金提供者的角度看，子交易对的资金调配，也即套利操作需要做如下考虑：

1）A池、B池必须都在套利操作中取得正向收益。一方收益、一方受损则有失公平，套利机制无法成立；

2）A池、B池从各自的角度考虑，都希望自己在套利操作中收益最大化。套利机制的理想目标是同时实现两个交易池双方的收益最大化。

3）A池、B池的套利收益需要合理平衡。一方收益大，另一方收益小，则套利机制也存在缺陷；

FreeSwap 交易协议的套利机制能够满足上述3点要求，即可以同时实现交易池双方的最大正向收益，并且双方受益相等。

套利操作的收益大小，可以通过套利操作前后，子交易对的 “恒定资产乘积” 的变化来衡量。下面对套利操作完成后，两个子交易对的 $K$ 值变化进行分析。

对于 $A$ 子交易对，套利后的K值变化为：

$$\begin{aligned}
{{\Delta}K}_A &= ( N_{AA} - L_A ) * ( N_B + L_B ) - N_{AA} * N_B \\[1mm]
&= N_{AA} * L_B - L_A * N_B - L_A * L_B \\[1mm]
&= {-P}_{A \rightarrow B}^e * L_A^2 + ( N_{AA} * P_{A \rightarrow B}^e - N_B ) * L_A \\[1mm]
&= {-P}_{A \rightarrow B}^e * \left( L_A - \frac{N_{AA} - N_B * P_{B \rightarrow A}^e}{2} \right)^2 + \frac{( N_{AA} - N_B * P_{B \rightarrow A}^e )^2}{4*P_{B \rightarrow A}^e}
\end{aligned} \tag{3.1.1} $$

可见对于 $A$ 池，当 $L_A = L_A^M$ 时，$K$ 值增加最大，即 $A$ 池获得最大套利收益：

$$\begin{aligned}
L_A^M &= \frac{N_{AA} - N_B * P_{B \rightarrow A}^e}{2} \\[3mm]
&= \frac{N_{AA}*N_{BB} - N_A*N_B}{2*(N_B + N_{BB})} \\
\end{aligned} \tag{3.1.2} $$

不难发现，关系式 $(2.4.19)$ 中的 $L_A^e$ 与 $L_A^M$ 相等，即 FreeSwap 的套利机制可以实现 $A$ 池的最大 $K$ 值增加: 

$$\begin{aligned}
{{\Delta}K}_A^M &= \frac{( N_{AA} - N_B * P_{B \rightarrow A}^e )^2}{4*P_{B \rightarrow A}^e} \\[3mm]
&= \frac{(N_{AA}*N_{BB} - N_A *N_B)^2}{4*(N_A+N_{AA})*(N_B+N_{BB})}
\end{aligned} \tag{3.1.3} $$

类似地，对于 $B$ 子交易对，套利后的K值变化为：

$$\begin{aligned}
{{\Delta}K}_B &= (N_{BB} - L_B) * (N_A + L_A) - N_{BB} * N_A \\[1mm]
&= N_{BB} * L_A - L_B * N_A - L_A * L_B \\[1mm]
&= {-P}_{B \rightarrow A}^e * L_B^2 + ( N_{BB} * P_{B \rightarrow A}^e - N_A ) * L_B \\[1mm]
&= {-P}_{B \rightarrow A}^e * \left( L_B - \frac{N_{BB} - N_A * P_{A \rightarrow B}^e}{2} \right)^2 + \frac{( N_{BB} - N_A * P_{A \rightarrow B}^e )^2}{4*P_{A \rightarrow B}^e} \end{aligned} \tag{3.1.4}$$

同样对于 $B$ 池，当 $L_B = L_B^M$ 时，$K$ 值有最大增加值，即 $B$ 池获得最大套利收益：

$$\begin{aligned}
L_B^M &= \frac{N_{BB} - N_A * P_{A \rightarrow B}^e}{2} \\[3mm]
&= \frac{N_{AA}*N_{BB} - N_A*N_B}{2*(N_A + N_{AA})} \\
\end{aligned} \tag{3.1.5} $$

同样，关系式 $(2.4.19)$ 中的 $L_B^e$ 与 $L_B^M$ 相等，即 FreeSwap 的套利机制可以实现 $B$ 池的最大 $K$ 值增加: 

$$\begin{aligned}
{{\Delta}K}_B^M &= \frac{( N_{BB} - N_A * P_{A \rightarrow B}^e)^2}{4*P_{A \rightarrow B}^e} \\[3mm]
&= \frac{(N_{AA}*N_{BB} - N_A *N_B)^2}{4*(N_A+N_{AA})*(N_B+N_{BB})}
\end{aligned} \tag{3.1.6} $$

比较 $(3.1.3)$ 、$(3.1.6)$ 以及 $(2.4.19)$，可以发现：

$$\Delta K_B^M \equiv \Delta K_A^M \equiv L_A^e*L_B^e \tag{3.1.7}$$

这意味着，FreeSwap 套利协议能够同时实现 $A$ 池、$B$ 池套利前后 $K$ 值增量最大化, 而且 $A$ 池、$B$ 池的 $K$ 值增量是完全相同的, 其值等于子资金池之间相互套利资产数量的乘积。

### 3.2 FreeSwap 套利收益率分析

为简化计算，这里假设 FreeSwap 交易及套利过程如下：

1）$A$ 池、$B$ 池初始具有相同数量的两种代币，其数量分别用 $X$, $Y$表示；

2）用户在 $A$ 池内用数量为 $x$ 的 $Token A$ , 兑换数量为 $y$ 的 $TokenB$ , 导致 $A$ 池资产价格滑动，滑动幅度为 $\gamma$ , 触发 $A$ 池、$B$ 池发生套利操作；

3）套利完成后，$A$ 池、$B$ 池的代币数量变为 $(X+x'_1\ ,\ Y-y'_1)$ 及 $(X+x'_2\ ,\ Y-y'_2)$；

交易及套利过程的 $Token$ 数量变化，如下表所示：

$$\begin{array}{lccc||c}
  子交易池：&  &  & A\ 池            & B\ 池 \\
  \hline
  代币状态：&  &  & (N_{AA}\ ,\ N_B) & (N_A\ ,\ N_{BB}) \\
  \hline
  初始转态：&  &  & (X\ ,\ Y) & (X\ ,\ Y) \\
  \hline
  兑换交易：&  &  & (X + x\ ,\ Y - y)  & (X\ ,\ Y) \\
  \hline
  完成套利：&  &  & (X + x'_1\ ,\ Y - y'_1) & (X +  x'_2\ ,\ Y - y'_2) \\
  \hline
 \end{array}$$

根据 "恒定资产乘积" 公式，有：
$$ (X+x)*(Y-y) = X*Y $$

$$ y = \frac{x}{X+x} * Y \tag{3.2.1} $$

根据 $(2.4.19)$ 中的套利触发条件，有：
$$ (X+x)*Y = \gamma * X* (Y-y) $$

$$ x = (\sqrt{\gamma} -1 )* X \tag{3.2.2} $$

根据 $(3.1.3)$ 和 $(3.1.6)$，子交易池套利后的最大 $K$ 值增加为：

$$\begin{aligned}
\Delta K_A^M = \Delta K_B^M &= \frac{((X+x)*Y-X*(Y-y))^2}{4*(X+(X+x))*((Y-y)+Y)} \\[3mm]
&= \frac{(x*Y+y*X)^2}{4*(2X+x)*(2Y-y)} \\[3mm]
&= \frac{(\sqrt{\gamma}-1)^2}{4\sqrt{\gamma}}*X*Y 
\end{aligned} \tag{3.2.3} $$

套利后，$A$ 池、$B$ 池的 $K$ 值增加比例为：
$$\delta K_A = \delta K_B = \frac{(\sqrt{\gamma}-1)^2}{4\sqrt{\gamma}} \tag{3.2.4} $$

根据 $(2.4.19)$，可以计算得出套利操作在子资金池之间相互交换的代币金额如下：

 $$ L_A^e \ = \ \frac{\ x \ }{2} \tag{3.2.5}$$

$$L_B^e \ = \ \frac{x}{2(X+x)} * Y = \ \frac{\ y \ }{2} \tag{3.2.6} $$

可见套利完成后，两个子资金池的代币数量变为：

$$ (X + \frac{\ x\ }{2}\ ,\ Y - \frac{\ y \ }{2} ) \  \ || \ \ (X + \frac{\ x\ }{2}\ ,\ Y - \frac{\ y \ }{2} ) \tag{3.2.7} $$

即两个子资金池的代币数量相等，达到完全的平衡。

上述分析，是基于 $A$ 池、$B$ 池的资金量完全相同的假设进行的，下面分析一下，如果 $A$ 池、$B$ 池的资金量不同，按照 FreeSwap 协议套利后， $A$ 池、$B$ 池的 $K$ 值增加比例。

此时，交易及套利过程的 $Token$ 数量变化，可如下表所示：

$$\begin{array}{lccc||c}
  子交易池：&  &  & A\ 池            & B\ 池 \\
  \hline
  代币状态：&  &  & (N_{AA}\ ,\ N_B) & (N_A\ ,\ N_{BB}) \\
  \hline
  初始转态：&  &  & (X_1\ ,\ Y_1) & (X_2\ ,\ Y_2) \\
  \hline
  兑换交易：&  &  & (X_1\ +\ x_1\ ,\ Y_1\ -\ y_1)  & (X_2\ ,\ Y_2) \\
  \hline
  完成套利：&  &  & (X_1\ +\ x_1'\ ,\ Y_1\ -\ y_1') & (X_2\ + \ x_2'\ ,\ Y_2\ - \ y_2') \\
  \hline
 \end{array}$$

 假设 $A$ 池、$B$ 池的资金比例用 $\beta$ 表示 , 则存在下面关系：

$$\left\{ \begin{array}{ccl}
  &\Large \frac{Y_1}{X_1} \normalsize = \Large \frac{Y_2}{X_2}  &(价格相同)    \\[3mm]
  &\Large \frac{X_2*Y_2}{X_1*Y_1} \normalsize = \beta    &(资金量比例)  \\[3mm]
  &(X_1 + x_1) * (Y_1 - y_1) = X_1 * Y_1            &(恒定乘积)  \\[3mm]
  &(X_1 + x_1) * Y_2 = \gamma * X_2 * (Y_1 -y_1)    &(触发套利)  \tag{3.2.8} \end{array} \right. $$

略去推导过程，经过计算可得：

$$\Delta K_A^M = \Delta K_B^M = \frac{(\gamma-1)^2}{4\gamma \Large (\normalsize\sqrt{\gamma} + \Large \frac{1}{\sqrt{\beta}})(\frac{1}{\sqrt{\gamma}} \normalsize + \Large \frac{1}{\sqrt{\beta}})} * X_1 * Y_1 
\tag{3.2.9} $$

套利后，$A$ 池、$B$ 池的 $K$ 值增加比例为：

$$\delta K_A = \frac{(\gamma-1)^2}{4\gamma \Large (\normalsize\sqrt{\gamma} + \Large \frac{1}{\sqrt{\beta}})(\frac{1}{\sqrt{\gamma}} \normalsize + \Large \frac{1}{\sqrt{\beta}})} \tag{3.2.10} $$

$$\delta K_B = \frac{(\sqrt{\gamma}-1)^2}{\ \ 4\sqrt{\gamma}(\sqrt{\beta\gamma}+1)(\sqrt{\beta}+\sqrt{\gamma}) \ \ \ } \tag{3.2.11} $$

由 $(3.2.10)$、$(3.2.11)$ 可见，$\delta K_A$ 随 $\beta$ 单调上升，而 $\delta K_B$ 随 $\beta$ 单调下降，这意味着，在两个子资金池资金量不平衡的情况下，资金池的资金量相对于另一个资金池越大，套利时，$K$ 值增加比例就越小，相反，资金池的资金量相对越小，套利时的 $K$ 值增加比例就越大。所以用户在加入流动性时，选择加入资金量较小的子资金池，会更为有利，这一内在调节机制可以使得 FreeSwap 交易协议的两个子交易池的资金量达到动态平衡。

### 3.3 等效交易费率

如果不是通过交易套利，而是通过收取交易手续费的方式，要取得与 $(3.2.4)$ 相同的 $K$ 值增加比例，交易手续费费率需要如何设置呢？

为表述方便，该交易费率用 $\alpha$ 表示。由于 FreeSwap 协议设有两个独立的单向交易资金池，考虑等效性，计算 $\alpha$ 时，应当考虑相同资金规模的单一双向交易池，该交易池可表示为：$(2X,2Y)$ 。用户利用与 $(3.2.2)$ 相同的数量为 $x$ 的 $Token A$ 换取一定数量的 $Token B$，设该数量为 $y'$，根据 “恒定资产乘积” 公式，有：

$$(2X+(1-\alpha)x) * (2Y-y') = 2X*2Y \tag{3.3.1} $$

交易完成后，由于收取的交易手续费未参与交易而是直接进入了资金池，导致整体资金池的 $K$ 值增加，该增加值为：

$$\begin{aligned}
\Delta K &= \alpha x *(2Y-y') \\
&= ax * \frac{2X*2Y}{2X+(1-\alpha)x}
\end{aligned} \tag{3.3.2} $$

相对于兑换交易发生前，整个资金池的 $K$ 值增加比例为：

$$\delta K = \frac{ax}{2X+(1-\alpha)x} \tag{3.3.3}$$

结合 $(3.2.2)$、$(3.2.4)$，令 $\delta K = \delta K_A$ ，有：

$$\alpha = \frac{\sqrt{\gamma}-1}{\sqrt{\gamma}+1} \tag{3.3.4}$$

计算可得, 当 $\gamma$ = 1.01 时，$\alpha ≈$ 2.488‰ , 即如果两个子交易池的价格偏离达到 1% 时，自动进行套利操作，资金池提供者的收益等价于收取交易用户 2.488‰ 的交易手续费。目前, UniSwap $^{[3]}$ 收取交易用户 3‰ 的交易手续费，FreeSwap 通过交易套利可以实现 UniSwap 大约 83% 的收益。考虑到 FreeSwap 协议能够为用户提供免费交易服务，可以吸引更多的交易用户，提升交易量，通过交易量的提升，在完全免除交易手续费的情况下，实现与 UniSwap 相当，甚至超过 UniSwap 的收益，是完全有可能的。

由 $(3.3.4)$ 也可导出：

$$\sqrt{\gamma} = \frac{\ 1+\alpha \ }{1-\alpha} \tag{3.3.5}$$

计算可得, 当 $\alpha$ = 3‰ 时，$\gamma ≈$ 1.0121‰ , 即 FreeSwap 协议如果在两个子交易池的价格偏离达到 1.21% 时进行套利操作，就可以取得与 UniSwap 的 3‰ 交易收费相同的收益。

### 3.4 不适用交易类型

由于 FreeSwap 交易协议依靠交易池价格偏离时的交易套利获得收益，因此 FreeSwap 不适用于两个交易对的代币均为稳定币的交易场景。稳定币的兑换交易需要尽量降低价格偏离，通过积累价格偏离，继而进行套利，不会有利于吸引稳定币的交易用户。

FreeSwap 交易协议也不适用于通缩型代币的交易对交易。由于套利操作会在两个子资金池之间进行代币互换，而通缩型代币的转账交易会造成代币通缩，不利于套利换入通缩型代币的一方子资金池，而且套利操作也会增加通缩型代币的转账交易次数，加速通缩，违背通缩型代币的设计初衷。

## 4. FreeSwap 协议总结

FreeSwap 交易协议以 “恒定资产乘积” 公式为基础，为每对数字资产创建两个单向交易的子资金池。方向相反的单向交易会造成两个子资金池的资金价格发生反向滑动，当价格滑动幅度达到设定数值时，两个子资金池自动进行相互套利操作，通过套利操作，为资金池提供者提供收益，同时恢复两个资金池资产价格的一致性。

FreeSwap 交易协议的套利机制能够同时实现两个子资金池的收益最大化，既保证了公平，又实现了双赢。计算表明，如果在资金池价差达到1% 时进行套利操作，FreeSwap 的套利交易机制可以在完全不收取交易手续费的情况下下，实现相当于收取 2.488 ‰ 交易手续费的收益。

FreeSwap 交易协议通过实现完全免费的去中心化交易机制，可以吸引更多的用户离开中心化交易所，以去中心化的方式参与加密资产投资。

## 参考文献

1、Vitalik Buterin, **Let's run on-chain decentralized exchanges the way we run prediction markets**, Dec. 2016. URL: [https://www.reddit.com/r/ethereum/comments/55m04x/lets_run_onchain_decentralized_exchanges_the_way](https://www.reddit.com/r/ethereum/comments/55m04x/lets_run_onchain_decentralized_exchanges_the_way).

2、 Hayden Adams, **Uniswap Whitepaper**. URL: [https://hackmd.io/C-DvwDSfSxuh-Gd4WKE_ig](https://hackmd.io/C-DvwDSfSxuh-Gd4WKE_ig).

3、 Hayden Adams et al, **Uniswap V2 Core**. URL: [https://uniswap.org/whitepaper.pdf](https://uniswap.org/whitepaper.pdf).

4、 Vitalik Buterin, **The x*y=k market maker model**. URL: [https://ethresear.ch/t/improving-front-running-resistance-of-x-y-k-market-makers](https://ethresear.ch/t/improving-front-running-resistance-of-x-y-k-market-makers).

5、Pintail, **Understanding Uniswap Returns**. URL: [https://medium.com/@pintail/understanding-uniswap-returns-cc593f3499ef](https://medium.com/@pintail/understanding-uniswap-returns-cc593f3499ef).

6、Pintail, **Uniswap: A Good Deal For Liquidity Providers?**. URL: [https://medium.com/@pintail/uniswap-a-good-deal-for-liquidity-providers-104c0b6816f2](https://medium.com/@pintail/uniswap-a-good-deal-for-liquidity-providers-104c0b6816f2).
