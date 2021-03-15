---
id: freeswap
title: FreeSwap Exchange Protocol 
hide_title: true
sidebar_label: FreeSwap Procotol
---

# FreeSwap Decentralized Exchange Protocol 

[TOC]

## Abstract 

FreeSwap protocol is a decentralized exchange protocol that does not charge any exchange fees. Based on the formula "constant-product invariant", FreeSwap creates two one-way-swap sub-pools for each pair of tokens. As the swap going, when the token price deviation in the two sub-pools reaches a certain extent, the sub-pools carry out arbitrage operation with each other. This arbitrage operation can rebalance the token prices in the sub-pools and make profits for liquidity providers. This paper mainly describes the FreeSwap arbitrage mechanism and its basic rules, expounds its win-win characteristics, and evaluates the arbitrage profit level quantitatively through a theoretical model. It is shown that FreeSwap can achieve the arbitrage profit equivalent to 2.488 ‰ swap fee while arbitraging at sub-pool price deviation of 1%.

## 0 Introduction

In 2020, "constant-product invariant" formula<sup>[1]</sup> made a great success in decentralized exchanges (DEXs). This formula is extremely concise, but it can automatically and perfectly determine the token price in the liquidity pools. DEXs based on the this formula, such as Uniswap<sup>[2][3]</sup> and SushiSwap, can provide competitive swapping liquidity and trading depth compared to centralized exchanges, and hav been attracting a large number of cryptocurrency investors.

However, in addition to paying the gas fee of blockchain network, DEX transactions also need to pay a certain percentage of exchange fees, which is some higher than centralized exchanges. The purpose of FreeSwap protocol is to design a completely free exchange protocol that does not require to pay any exchange fees. It is expected that by removing exchange fees, more and more users can be attracted to join DEXs and leave centralized exchanges.

To solve the problem of "front running attack", Vitalik Buterin once proposed to set up two one-way trading sides for a token pair<sup>[4]</sup>. FreeSwap protocol refines his proposal and defines the implementation details. Since the two one-way trading pools can only exchange tokens in one direction, this will inevitably lead to the reverse deviation of token prices between two trading pools. This deviation can not only stop "front-running attack" of miners, but also provide a way to make profits for liquidity providers. FreeSwap protocol makes full use of the arbitrage opportunities coming from the price deviations between the two sub-pools to provide returns to liquidity providers, and further, to serve the exchange users completely with no trade fee.

## 1 Basics of liquidity pool

### 1.1 Swap pair definition

The swap pair consists of two different tokens that can be exchanged, represented by $Token A$ and $Token B$, respectively. When creating a swap pair, the user needs to deposit corresponding number of $Token A$ and $Token B$ with equal value according to the actual market price. If the price differs from the market price, the pair will be arbitraged and the creator will suffer a loss.

Assuming $P_{A}$ and $P_{B}$ are respectively the price of $Token A$ and $Token B$ in any legal currency, $P_{A\rightarrow B}$ is defined as the price exchanging $Token A$ for $Token B$, which is the amount of $Token B$ equivalent to one $Token A$ in value:

$$
P_{A \rightarrow B} = \frac{P_A}{P_B} \tag{1.1.1}
$$

Similarly, define $P_{B \rightarrow A}$, which is the price at which $Token B$ is traded to $Token A$:

$$
P_{B \rightarrow A} = \frac{P_B}{P_A} \tag{1.1.2}
$$

In theory, without considering trading costs, the following relationship exists:

$$
P_{A \rightarrow B} * P_{B \rightarrow A} = 1 \tag{1.1.3}
$$

Assuming $N_A$ and $N_B$ are the quantities of two digital assets $Token A$ and $Token B$ at any time within the trade pair, we express the trade pair as:

$$
(N_A \ | \ N_B) \tag{1.1.4}
$$

The trade pair always assume two internal tokens have equal market values all the time, so following formula are equivalent:

$$
N_A * P_{A} = N_B * P_{B} \tag{1.1.5}
$$

$$
N_A = N_B * P_{B \rightarrow A} \tag{1.1.6}
$$ 

$$
N_B = N_A * P_{A \rightarrow B} \tag{1.1.7}
$$

$$
P_{A \rightarrow B} = \frac{N_B}{N_A} \tag{1.1.8}
$$

$$
P_{B \rightarrow A} = \frac{N_A}{N_B} \tag{1.1.9}
$$

### 1.2 Constant-product invariant

While performing token exchange, since the external prices of $Token A$ and $Token B$ cannot be easily obtained, it is needed to design a mechanism to determine the exchange ratio between $Token A$ and $Token B$. The DEXs of automated market maker (AMM) type use the "constant-product invariant" formula<sup>[1]</sup> to determine the change of the number of tokens in the trading pool before and after the exchange, also the amount of tokens traded in and out accordingly.

For the pair of $(N_A \ | \ N_B)$, "constant-product invariant" formula is represented as:

$$
N_A * N_B=K \tag{1.2.1}
$$

In this formula, the value of $K$ changes only when the users deposit tokens into, or remove tokens from the trading pair, and alway remains constant during the trading process.

Suppose in an exchange oparetion, the user trades $Token A$ for $Token B$, and the input amount of $Token A$ is $\mathrm{\Delta}_{A}$, the output amount of $Token B$ is $\mathrm{\Delta}_{B}$. ( "Trade in" and "trade out" here after are from the point of view of liquidity pool, if from the point of users, the relationship "trade in" and "trade out" is completely opposite). According to the "constant-product invariant" formula, there are:

$$
( N_A + \mathrm{\Delta}_A ) * (N_B - \mathrm{\Delta}_B ) = ( N_A * N_B )  \tag{1.2.2}
$$

$$
\mathrm{\Delta}_B = \frac{\mathrm{\Delta}_A}{N_A + \mathrm{\Delta}_A} * N_B \tag{1.2.3}
$$

It can be seen that there are three different prices here, before the trade, after the trade, and happend during the trade.

The token price of the pool before the trade (expressed as $P_{A \rightarrow B}^{0}$):

$$
P_{A \rightarrow B}^{0} = \frac{N_B}{N_A} \tag{1.2.4}
$$

The actual token price taken in the trade (represented as $P_{A \rightarrow B}^{1}$) is:

$$
P_{A \rightarrow B}^{1} = \frac{\mathrm{\Delta}_B}{\mathrm{\Delta}_A} = \frac{N_B}{N_A + \mathrm{\Delta}_A} \tag{1.2.5}
$$

The token price after the trade finished (represented as $P_{A \rightarrow B}^{2}$) is:

$$
P_{A \rightarrow B}^{2} = \frac{N_B - \mathrm{\Delta}_B}{N_A +  \mathrm{\Delta}_A} \tag{1.2.6}
$$

Obviously:

$$
P_{A \rightarrow B}^{0} > P_{A \rightarrow B}^{1} > P_{A \rightarrow B}^{2} \tag{1.2.7}
$$

It means, the price of trade-in token $Token A$ relative to the trade-out token $Token B$ has slipped after the trade has finished. The price of $Token A$ in the pair before the trade starts is higher than the actual price the trade adopts, and the actual price the trade adopts is higher than the price of $Token A$ in the pair after the trade is completed. That is, after the token trade is completed, the price of $Token A$ relative to $Token B$ in the pair has fallen, and accordingly, the relative price of $Token B$ has risen.

### 1.3 Trade arbitrage

The exchange operation happens at the moment the transaction block is confirmed on the blockchain. The external prices of $Token A$ and $Token B$ actually do not change at that moment. Due to the constraint of "constant-product invariant", the token price will slide within the exchange, which will cause the exchange user to suffer a certain marginal exchange loss. The exchange loss in the amount of $Token B$ is calculated as follows:

$$\begin{aligned} Lost_{A \rightarrow B} &= \mathrm{\Delta}_A * \frac{N_B}{N_A} - \mathrm{\Delta}_A * \frac{N_B}{N_A + \mathrm{\Delta}_A} \\[3mm]
&= \mathrm{\Delta}_A * \frac{N_B}{N_A} \Large{/} \normalsize ( 1 + \frac{N_A} {\mathrm{\Delta}_{A}} ) \end{aligned} \tag{1.3.1}$$

It can be seen that the greater the ratio $(\mathrm{\Delta}_{A} / N_A)$ the user trade $Token A$ into the pool, the greater the exchange loss he/she will suffer.

After the exchange is completed, if another user performs reverse exchange, according to the "constant-product invariant" formula, he/she only needs to pay $Token B$ in the amount of $\mathrm{\Delta}_B$, and can get $Token A$ of $\mathrm{\Delta}_A$. After this reverse exchange, the asset price in the pool will return to the initial price $P_{A \rightarrow B} = N_B/N_A$. That is, this user completes the exchange at the higher $Token B$ to $Token A$ price, and commits the arbitrage of the previous user's exchange loss, the arbitrage profit is equal to the previous user's exchange loss.

### 1.4 Impermanent loss

When providing liquidity, the provider needs to deposit into the trading pool two kinds of tokens with the corresponding amounts of same total value. With the trade on going, the number of tokens corresponding to the user's liquidity will change. Assuming that the user’s initial investment of $Token A$ is $X_{A}$, when the user withdraws the liquidity, if the market price of $Token A$ rises relative to $Token B$, then ${X'}_{A}$，the amount of $Token A$ that the user can get from the pool may be less than $X_{A}$. If the liquidity service income obtained by providing liquidity are not enough to compensate for the loss caused by the decrease in the amount of $Token A$, in this case, the liquidity provider will suffer losses relative to simply holding $Token A$ and $Token B$ instead of providing liquidity services. In fact, no matter whether the price of $Token A$ and $Token B$ goes up or down, as long as their price fluctuates from the time providing liquidity, the user will always suffer losses. The amount of the loss is completely determined by the market price variation, beyond any user's control, which is also called "Impermanent Loss"<sup>[5]</sup>.

Taking $Token B$ as the price benchmark, following quantitative analysis of impermanent loss is conducted without considering the exchange fee income. Assuming that the initial number of tokens that the user puts into the trade pair are $(X_A|Y_B)$, the price of $Token A$ relative to $Token B$ is $P_1$, and at the end, the number of tokens that the user withdraws from the trade pair are $(X'_A|Y'_B)$, and the price of $Token A$ relative to $Token B$ is $P_2$, if the user does not provide liquidity for the trading pool, but simply holds $(X_A|Y_B)$ tokens, the final value is:

$$
V_1=X_A*P_2+Y_B \tag{1.4.1}
$$

Since the user provides liquidity for the trading pool, the actual value of $(X'_{A}|Y'_{B})$ tokens is:

$$
V_2=X'_A*P_2+Y'_B \tag{1.4.2}
$$

Considering:

$$\left\{ \begin{matrix} \begin{aligned}
X_A * Y_B &= X'_A * Y'_B \\
P_1 &= Y_B / X_A \\
P_2 &= Y'_B / X'_A \\
\end{aligned} \end{matrix} \right. \tag{1.4.3}$$

Then we have<sup>[6]</sup>:

$$\begin{aligned} \frac{V_2}{V_1} &= \frac{\ 2\sqrt{P_1*P_2}\ }{P_1+P_2} \\[3mm]
&= \sqrt{\frac{4P_{1}P_{2}}{( P_{1} - P_{2} )^{2} + 4P_{1}P_{2}}} \leq 1 \end{aligned} \tag{1.4.4}$$

So, as long as $P_1 \neq P_2$, $V_2$ is always less than $V_1$, that is to say, the users's total token value denominated in $Token B$ is definitely reduced compared to simply holding two tokens. Similarly the total token value denominated in $Token A$ is also definitely reduced compared to simply holding two tokens. Therefore, the so-called "Impermanent Loss" is actually a permanent loss that is bound to occur, only the amount of loss may change with the price ratio.

## 2 FreeSwap exchange protocol

### 2.1 FreetSwap protocol goal

The goal of FreeSwap protocol is to implement a decentralized exchange protocol with no trade fees at all. 

The "constant-product invariant" mechanism inevitably leads to token price slippage, from which FreeSwap could make profits for liquidity provider by arbitraging automatically.

### 2.2 FreeSwap trade pair setup

FreeSwap trade protocol sets up two independent one-way sub-pools for the token pair `(Token_A|Token_B)`, expressed as:

$$
( N_{AA} | N_B)\ || \ (N_A | N_{BB} ) \tag{2.2.1} 
$$

Where, the sub-pool $(N_{AA}|N_B)$ (called as A-Pool hereinafter) can trade in $Token A$, trade out $Token B$, $N_{AA}$ is the number of $Token A$, and $N_B$ is the number of $Token B$ in A-Pool.

The sub-pool $(N_A|N_{BB})$ (called as B-Pool hereinafter) can trade in $Token B$, trade out $Token A$, $N_{BB}$ is the number of $Token B$, and $N_A$ is the number of $Token A$ in B-Pool.

When the liquidity provider adds tokens into the trading pool, he can specify which sub-pool to add, or he can add into both the sub-pools at the same time with specified ratio.

### 2.3 FreetSwap token exchange

When the user trades $Token A$ and $Token B$ with the trading pool, there are two opposite trading operations, one is to trade in $Token A$, and trade out $Token B$; the other is to trade in $Token B$, and trade out $Token A$. Here, "trade in" and "trade out" are relative to the trading pool.

If the user trades $Token A$ for $Token B$, FreeSwap will pass this trade to A-Pool, the $(N_{AA}|N_B )$ sub-pool to process. The A-Pool always trades in $Token A$ and trades out $Token B$, so the number of $N_{AA}$ always goes up, and the number of $N_B$ always goes down, which will lead to the monotonous decrease of $Token A$ price denominated by $Token B$, and finally resulting $Token A$ price deviating from actual market price. 

Similarly, if the user trade in $Token B$ for $Token A$, B-Pool will handle this trade. B-Pool always trades in $Token B$, and trades out $Token A$, so the amount of $N_{BB}$ will keep rising, and the amount of $N_A$ keep falling, which will also cause that $B$ price denominated in $Token A$ decreases monotonically and eventually deviates from the actual market price.

The sub-pool $(N_{AA}|N_B)$ and $(N_A | N_{BB} )$ are two one-way trading pairs in opposite directions, they are always fixed at one token in and the other token out. As the trades happening, the prices of the pair tokens always slide in opposite directions. After some trade volume accumulation, the price difference of the same token in the two sub-pools will exceed a certain level, which is too large to prevent users from participating the exchange. At this time, it is necessary to internally exchange the tokens between the sub-pools to smooth the token price difference. The process of internal token exchange within the sub-pools is actually a arbitrage process. As trading users trade with the two one-way trading pools, token prices will slide in opposite ways, and consequently users will suffer trade losses becasue of this price slippage. By executing the internal exchange between two sub-pools, price slippage can be fixed, and the arbitrage to user trade slippage losses can be achieved, which may make profits for the liquidity providers of the pool.

### 2.4 FreeSwap exchange arbitrage

As said above, FreeSwap sets up two independent sub-trading pools:

$$
( N_{AA} | N_B )\ || \ ( N_A | N_{BB} ) \tag{2.4.1}
$$

The prices of $Token A$ and $Token B$ in the two sub-pools will deviate as exchange occurs. When the price deviates to some extent, the two sub-pools need to internally exchange tokens to restore the token price. The token exchange between two sub-pools can restore the token prices of both side, and coulde also commit some kind of arbitrage between the pools. The analysis following:

Before arbitrage, $Token A$ prices in two pools are:

$$
P_{AA \rightarrow B} = \frac{N_B}{N_{AA}} \tag{2.4.2}
$$

$$
P_{A \leftarrow {BB}} = \frac{N_{BB}}{N_A} \tag{2.4.3}
$$

The deviation ratio is:

$$
R_{PA} = \frac{P_{A \leftarrow BB}}{P_{AA \rightarrow B}} = \frac{N_{AA} * N_{BB}}{N_B \ * \ N_A} \tag{2.4.4}
$$

Similarly, $Token B$ prices in the two pools are:

$$
P_{BB \rightarrow A} = \frac{N_A}{N_{BB}} \tag{2.4.5}
$$

$$
P_{B \leftarrow {AA}} = \frac{N_{AA}}{N_B} \tag{2.4.6}
$$

The deviation ratio is:

$$
R_{PB} = \frac{P_{B \leftarrow AA}}{P_{BB \rightarrow A}} = \frac{N_{AA} * N_{BB}}{N_B * N_A} \tag{2.4.7}
$$

It can be seen that the price ratios of $Token A$ and $Token B$ in the two sub-pool are the same, namely:

$$
R_{PA} = R_{PB} = R_P = \frac{N_{AA} * N_{BB}}{N_B * N_A} \tag{2.4.8}
$$

The FreeSwap protocol specifies that when the token prices of two sub-pool deviate to a certeain threshold $\gamma$, such as the value of $R_{P}$ is greater than 101%, the internal arbitrage exchange is automatically executed to restore the token price.

Arbitrage operation is to exchange one token with more amount in one sub-pool for the other token with more amount in the other sub-pool with the equivalent value, that is, $(N_{AA}|N_B)$ sub-pool swap $Token A$ for $Token B$ with same value in $(N_A|N_{BB})$. From another perspective, it is for $(N_A|N_{BB})$ sub-pool to exchange $Token B$ for $Token A$ in $(N_{AA}|N_B )$ sub-pool with equal value.

To ensure the exchange is fair, the exchange price is set as the average price of all tokens in the two sub-pools, namely:

$$
P_{A \rightarrow B}^e = \frac{N_B + N_{BB}}{N_A + N_{AA}} \tag{2.4.9}
$$

$$
P_{B \rightarrow A}^e = \ \frac{N_A + N_{AA}}{N_B + N_{BB}} \tag{2.4.10}
$$

$$
P_{A \rightarrow B}^e * P_{B \rightarrow A}^e = \ 1 \tag{2.4.11}
$$

After the arbitrage completed, the two sub-pools can be expressed as:

$$
(N_{AA} - L_A \ | \ N_B + L_B  )\ ||\ ( N_A + L_A \ | \ N_{BB} - L_B ) \tag{2.4.12}
$$

That is, in this trade arbitrage, A-Pool exchanges $Token A$ with a quantity of $L_A$ for $Token B$ with a quantity of $L_B$. Considering the exchange value is equal, the following equation exists:
 
$$
L_{B} = L_{A}*P_{A \rightarrow B}^e \tag{2.4.13}
$$

After the arbitrage operation, $Token A$ price of the A-Pool (priced in $Token B$) will increase, but it should not exceed the average price of the overall trading pool $P_{A \rightarrow B}^e$ for the reason of fairness. Similarly, $Token B$ price (priced in $Token A$) in B-Pool will also rise, but also should not exceed the average price $P_{B \rightarrow A}^e$ , That is, the following relationship exists:

$$
\frac{N_B + L_B}{N_{AA} - L_A} \leq P_{A \rightarrow B}^e \tag{2.4.14} 
$$

$$
\frac{N_A \ + \ L_A}{N_{BB} \ - \ L_B} \normalsize \leq P_{B \rightarrow A}^e \tag{2.4.15}
$$

Combining $(2.4.13)-(2.4.15)$, following relationships can be derived: 

$$
L_A \leq \frac{N_{AA}*N_{BB} - N_A*N_B}{2*(N_B + N_{BB})}  \tag{2.4.17} 
$$

$$
L_B \leq \frac{N_{AA}*N_{BB} - N_A*N_B}{2*(N_A + N_{AA})}  \tag{2.4.18} 
$$

In order to reduce the times of arbitrage operation, arbitrage between two sub-pools should exchange as many tokens as possible to smooth token price deviations to the greatest extent. So $L_A$, $L_B$ should take the maximum vaue given by $(2.4.17)$ and $(2.4.18)$. It is easy to find, in this case, the token prices of the two sub-pools both happen to be restored to the average token price of the whole trade pool $P_{A \rightarrow B}^e$, which is also the price at which the two sub-pools exchange tokens.

In general, FreeSwap arbitrage can be expressed as follows:

$$\left\{ \begin{aligned} 
\ \ &\frac{N_{AA} * N_{BB}}{N_A * N_B} \ \geq \ \gamma \\[3mm]
P_{A \rightarrow B}^e \ = \ \ &\frac{N_B + N_{BB}}{N_A + N_{AA}} \\[3mm]
L_A^e \ = \ \ &\frac{N_{AA}*N_{BB} - N_A*N_B}{2*(N_B + N_{BB})}  \\[3mm]
L_B^e \ = \ \ &\frac{N_{AA}*N_{BB} - N_A*N_B}{2*(N_A + N_{AA})}  \\[3mm]
\end{aligned} \right. \tag{2.4.19} $$ 

Among the above equation, $\gamma$ is the condition for triggering the arbitrage operation, which means that arbitrage is executed when the price deviation of the two sub-pools is greater than or equal to $\gamma-1$.  $L_A^e$, $L_B^e$ are the amount of tokens exchanged between the two sub-pools within the arbitrage operation. Arbitrage is taken by exchanging $L_A^e$, $L_B^e$ amount of `TokenA`, $Token B$ at the price of $P_{A \rightarrow B}^e$, on arbitrage completion, the token prices of the two sub-pools are exactly the same, both are $P_{A \rightarrow B}^e$.

## 3 FreeSwap arbitrage analysis

### 3.1 FreeSwap arbitrage analysis

From the perspective of the liquidity provider, for the internal exchange of the two sub-pools, ie. arbitrage operations, following considerations should be taken:

  1. Pool A and Pool B must both obtain positive returns in arbitrage operations. If one party gains and the other loses, the arbitrage mechanism does not hold;

  2. The arbitrage return of two sub-pools should be balanced reasonably. If one side gains more and the other side gains less, the arbitrage mechanism is also not perfect.

  3. Pool A and Pool B both expect to maximize their profits in the arbitrage operations. The ideal goal of the arbitrage mechanism is to maximize the profits of both sub-pools simultaneously.

FreeSwap arbitrage protocol can meet the above three requirements, that is, it can achieve the maximum positive profit for both sides of the trading pools, and the profits of both sides are equal.

The arbitrage profit can be counted by the changes of "constant-product invariant" of the sub-pools before and after the arbitrage operation. After the arbitrage completed, the changes in the $K$ value of the two sub-pools are analyzed below.

For the A-Pool, the change in K value after arbitrage is:

$$\begin{aligned}
{{\Delta}K}_A &= ( N_{AA} - L_A ) * ( N_B + L_B ) - N_{AA} * N_B \\[1mm]
&= N_{AA} * L_B - L_A * N_B - L_A * L_B \\[1mm]
&= {-P}_{A \rightarrow B}^e * L_A^2 + ( N_{AA} * P_{A \rightarrow B}^e - N_B ) * L_A \\[1mm]
&= {-P}_{A \rightarrow B}^e * \left( L_A - \frac{N_{AA} - N_B * P_{B \rightarrow A}^e}{2} \right)^2 + \frac{( N_{AA} - N_B * P_{B \rightarrow A}^e )^2}{4*P_{B \rightarrow A}^e}
\end{aligned} \tag{3.1.1} $$

It can be seen that for A-Pool, while $L_A = L_A^M$, $K$ value increases the most, that is, A-Pool gets the maximum arbitrage profit as:

$$\begin{aligned}
L_A^M &= \frac{N_{AA} - N_B * P_{B \rightarrow A}^e}{2} \\[3mm]
&= \frac{N_{AA}*N_{BB} - N_A*N_B}{2*(N_B + N_{BB})} \\
\end{aligned} \tag{3.1.2} $$

It is obvious that $L_A^e$ in $(2.4.19)$ equals to $L_A^M$, it means that FreeSwap arbitrage mechanism can increase $K$ value of the A-Pool to its maximum, which is:

$$\begin{aligned}
{{\Delta}K}_A^M &= \frac{( N_{AA} - N_B * P_{B \rightarrow A}^e )^2}{4*P_{B \rightarrow A}^e} \\[3mm]
&= \frac{(N_{AA}*N_{BB} - N_A *N_B)^2}{4*(N_A+N_{AA})*(N_B+N_{BB})}
\end{aligned} \tag{3.1.3} $$

Similarly, for the B-Pool, the K value changes after arbitrage is as follows:

$$\begin{aligned}
{{\Delta}K}_B &= (N_{BB} - L_B) * (N_A + L_A) - N_{BB} * N_A \\[1mm]
&= N_{BB} * L_A - L_B * N_A - L_A * L_B \\[1mm]
&= {-P}_{B \rightarrow A}^e * L_B^2 + ( N_{BB} * P_{B \rightarrow A}^e - N_A ) * L_B \\[1mm]
&= {-P}_{B \rightarrow A}^e * \left( L_B - \frac{N_{BB} - N_A * P_{A \rightarrow B}^e}{2} \right)^2 + \frac{( N_{BB} - N_A * P_{A \rightarrow B}^e )^2}{4*P_{A \rightarrow B}^e} \end{aligned} \tag{3.1.4}$$

Similarly for the B-Pool, when $L_B = L_B^M$, the $K$ value has the largest increase, that is, the B-Pool gets the maximum arbitrage profit:

$$\begin{aligned}
L_B^M &= \frac{N_{BB} - N_A * P_{A \rightarrow B}^e}{2} \\[3mm]
&= \frac{N_{AA}*N_{BB} - N_A*N_B}{2*(N_A + N_{AA})} \\
\end{aligned} \tag{3.1.5} $$

Samely, $L_B^e$ in $(2.4.19)$ is equal to $L_B^M$, that is, FreeSwap arbitrage mechanism can increase $K$ value of B-Pool to the maximum:

$$
\begin{aligned}
{{\Delta}K}_B^M &= \frac{( N_{BB} - N_A * P_{A \rightarrow B}^e)^2}{4*P_{A \rightarrow B}^e} \\[3mm]
&= \frac{(N_{AA}*N_{BB} - N_A *N_B)^2}{4*(N_A+N_{AA})*(N_B+N_{BB})}
\end{aligned} \tag{3.1.6} 
$$

Comparing $(3.1.3)$, $(3.1.6)$ and $(2.4.19)$, we can see that:

$$
\Delta K_B^M \equiv \Delta K_A^M \equiv L_A^e*L_B^e \tag{3.1.7}
$$

This means that FreeSwap arbitrage protocol can simultaneously maximize the increment of $K$ Value both for A-Pool and B-Pool by arbitrage, and the $K$ value increment in the two sub-pools are exactly the same, which equals to the product of the amount of tokens internaly exchanged within arbitrage between the sub-pools.

### 3.2 FreeSwap arbitrage profit

To simplify the calculation, it is assumed that the FreeSwap trading and arbitraging process is as follows:

  1. The A-Pool and the B-Pool initially have the same amount of two tokens, which are represented by $X$ and $Y$ respectively;

  2. In A-Pool, the user exchange $Token A$ with a quantity of $x$ for `TokenB` with a quantity of $y$, causing the token price of the A-Pool to slide. And the sliding value is $\gamma$, triggering arbitrage operation between A-Pool and B-Pool;

  3. After arbitrage, the number of tokens in the A-Pool and the B-Pool becomes $(X+x'_1\ ,\ Y-y'_1)$ and $(X+x'_2\ ,\ Y -y'_2)$；

The changes in the amount of $Tokens$ during the trade and arbitrage process are shown in the following table:

$$\begin{array}{lcc|c||c}
  Sub\ Pool: &  &  & A-Pool            & B-Pool \\
  \hline
  Token\ State: &  &  & (N_{AA}\ ,\ N_B) & (N_A\ ,\ N_{BB}) \\
  \hline
  Initial\ State: &  &  & (X\ ,\ Y) & (X\ ,\ Y) \\
  \hline
  Post\ Trade: &  &  & (X + x\ ,\ Y - y)  & (X\ ,\ Y) \\
  \hline
  Post Arbitrage: &  &  & (X + x'_1\ ,\ Y - y'_1) & (X +  x'_2\ ,\ Y - y'_2) \\
  \hline
 \end{array}$$

According to the "constant-product invariant" formula, there are:

$$
 (X+x)*(Y-y) = X*Y 
$$

$$
 y = \frac{x}{X+x} * Y \tag{3.2.1} 
 $$

According to the arbitrage triggering conditions in $(2.4.19)$, there are:

$$
 (X+x)*Y = \gamma * X* (Y-y) 
 $$

$$
 x = (\sqrt{\gamma} -1 )* X \tag{3.2.2} 
 $$

According to $(3.1.3)$ and $(3.1.6)$, after arbitrage the maximum $K$ value increasement of two sub-pools are:

$$\begin{aligned}
\Delta K_A^M = \Delta K_B^M &= \frac{((X+x)*Y-X*(Y-y))^2}{4*(X+(X+x))*((Y-y)+Y)} \\[3mm]
&= \frac{(x*Y+y*X)^2}{4*(2X+x)*(2Y-y)} \\[3mm]
&= \frac{(\sqrt{\gamma}-1)^2}{4\sqrt{\gamma}}*X*Y 
\end{aligned} \tag{3.2.3} $$

After arbitrage, the increase ratio of $K$ value both in A-Pool and B-Pool will be:

$$
\delta K_A = \delta K_B = \frac{(\sqrt{\gamma}-1)^2}{4\sqrt{\gamma}} \tag{3.2.4} 
$$

According to $(2.4.19)$, the amount of tokens exchanged between the two sub-pools during arbitrage operation can be calculated as follows:

$$
 L_A^e \ = \ \frac{\ x \ }{2} \tag{3.2.5}
$$

$$
L_B^e \ = \ \frac{x}{2(X+x)} * Y = \ \frac{\ y \ }{2} \tag{3.2.6} 
$$

It can be seen that after arbitrage, the number of tokens in the two sub-pools becomes:

$$
 (X + \frac{\ x\ }{2}\ ,\ Y - \frac{\ y \ }{2} ) \  \ || \ \ (X + \frac{\ x\ }{2}\ ,\ Y - \frac{\ y \ }{2} ) \tag{3.2.7} 
$$

That is, the number of tokens in the two sub-pools is equal, achieving a complete balance.

The above analysis is based on the assumption that the amount of tokens in A-Pool and the B-Pool are exactly the same. Let’s analyze below, if the amounts of tokens in A-Pool and the B-Pool are different, how much is the $K$ value increase of the two sub-pools after arbitrage according to FreeSwap protocol. 

In this case, the changes in the amount of tokens during the trade and arbitrage process are shown in the following table:

$$
  \begin{array}{lccc||c}
  Sub Pools: &  &  & A-Pool            & B-Pool \\
  \hline
  Token\ State: &  &  & (N_{AA}\ ,\ N_B) & (N_A\ ,\ N_{BB}) \\
  \hline
  Initial\ State: &  &  & (X_1\ ,\ Y_1) & (X_2\ ,\ Y_2) \\
  \hline
  Post\ Trade: &  &  & (X_1\ +\ x_1\ ,\ Y_1\ -\ y_1)  & (X_2\ ,\ Y_2) \\
  \hline
  Post Arbitrage: &  &  & (X_1\ +\ x_1'\ ,\ Y_1\ -\ y_1') & (X_2\ + \ x_2'\ ,\ Y_2\ - \ y_2') \\
  \hline
 \end{array}
$$

  Assuming that the $K$ Value ratio of A-Pool and B-Pool is represented by $\beta$, the following relationship exists:

$$
\left\{ \begin{array}{ccl}
  &\Large \frac{Y_1}{X_1} \normalsize = \Large \frac{Y_2}{X_2}  & \small (Same\ price)    \\[3mm]
  &\Large \frac{X_2*Y_2}{X_1*Y_1} \normalsize = \beta    &\small  (K\ value\ ratio)  \\[3mm]
  &(X_1 + x_1) * (Y_1 - y_1) = X_1 * Y_1            &\small  (Constant\ invariant)  \\[3mm]
  &(X_1 + x_1) * Y_2 = \gamma * X_2 * (Y_1 -y_1)    &\small  (Trigger\ arbitrage)  \tag{3.2.8} \end{array} \right. 
$$

Jumping the dedcution process, it can be concluded that:

$$
\Delta K_A^M = \Delta K_B^M = \frac{(\gamma-1)^2}{4\gamma \Large (\normalsize\sqrt{\gamma} + \Large \frac{1}{\sqrt{\beta}})(\frac{1}{\sqrt{\gamma}} \normalsize + \Large \frac{1}{\sqrt{\beta}})} * X_1 * Y_1 
\tag{3.2.9} 
$$

So after arbitrage, the K value increase ratio of A-Pool and B pool are:

$$
\delta K_A = \frac{(\gamma-1)^2}{4\gamma \Large (\normalsize\sqrt{\gamma} + \Large \frac{1}{\sqrt{\beta}})(\frac{1}{\sqrt{\gamma}} \normalsize + \Large \frac{1}{\sqrt{\beta}})} \tag{3.2.10} 
$$

$$
\delta K_B = \frac{(\sqrt{\gamma}-1)^2}{\ \ 4\sqrt{\gamma}(\sqrt{\beta\gamma}+1)(\sqrt{\beta}+\sqrt{\gamma}) \ \ \ } \tag{3.2.11} 
$$

It can be seen from $(3.2.10)$ and $(3.2.11)$ that $\delta K_A$ increases monotonically with $\beta$, while $\delta K_B$ decreases monotonically with $\beta$, which means that when the amount of funds in the two sub-pools is unbalanced, the more the amount of tokens in the sub-pool relative to the other one, the less the increase in $K$ value during arbitrage. On the contrary, the less the amount of tokens in the sub-pool, the more the increase in $K$ value during arbitrage. Therefore, when users add liquidity, it is more advantageous to choose to join the sub-pool with smaller amount of tokens, and on the round it helps improve the liquidity of this smaller pool. This internal regulation mechanism of FreeSwap protocol can make the amount of tokens in the two sub-pools to keep in a dynamic balance.

### 3.3 Equivalent transaction fee rate

Assuming not by arbitraging, but by charging exchange fees, how much exchange fee rate should be charged to obtain the same $K$ value increase as $(3.2.4)$?

Let's represent this trade fee rate as $\alpha$. Since the FreeSwap protocol has two independent one-way sub-pools, considering the equivalence, when calculating $\alpha$, the single two-way trade pool should have the same token amount, which can be expressed as: $(2X,2Y)$. The user exchanges $Token A$ with the same amount of $x$ as $(3.2.2)$ for some amount of $Token B$, denoted by $y'$. According to the "constant-product invariant" formula, there are:

$$
(2X+(1-\alpha)x) * (2Y-y') = 2X*2Y \tag{3.3.1} 
$$

After the completion of the exchange, because the exchange fees do not participate in "constant-product invariant" calculation, just directly enter the liquidity pool, resulting in an increase in the $K$ value of the overall token pool:

$$\begin{aligned}
\Delta K &= \alpha x *(2Y-y') \\
&= \alpha x * \frac{2X*2Y}{2X+(1-\alpha)x}
\end{aligned} \tag{3.3.2} $$

Compared with the $K$ value before the exchange, the $K$ Value increase of the whole pool is as follows:

$$
\delta K = \frac{\alpha x}{2X+(1-\alpha)x} \tag{3.3.3}
$$

Combining with $(3.2.2)$, $(3.2.4)$,  let $\delta K = \delta K_A$, we have:

$$
\alpha = \frac{\sqrt{\gamma}-1}{\sqrt{\gamma}+1} \tag{3.3.4}
$$

As the calculation example, when $\gamma$ = 1.01, $\alpha ≈$ 2.488‰, that is, if the arbitrage is automatically conducted while the price deviation of the two sub-pools reaches 1%, the profit of the liquidity pool provider is equivalent to charge exchange fee of 2.488‰ from trading users. Currently, UniSwap<sup>[2][3]</sup> charges an exchange fee of 3‰ from trading users, so FreeSwap can achieve approximately 83% of UniSwap's profit rate through only exchange arbitrage. Taking into account that the FreeSwap protocol can provide users with totally free exchange, it can attract more exchange users and increase exchange volume. By the increase in exchange volume, it is possible for FreeSwap to achieve the same profit level as UniSwap, or even exceed UniSwap while completely waiving any exchange fees. 

It can also be deduced from $(3.3.4)$:

$$
\sqrt{\gamma} = \frac{\ 1+\alpha \ }{1-\alpha} \tag{3.3.5}
$$

According to the calculation, $\gamma ≈$ 1.0121‰ while $\alpha$ = 3‰,  that is, if FreeSwap performs arbitrage when the price deviation of the two sub-pools reaches 1.21%, it can obtain equivalent 3‰ of exchange fee income same as UniSwap.

### 3.4 Not applicable exchanges

Since FreeSwap protocol relies on exchange arbitrage when the price of sub-pools deviate to a certain level, FreeSwap is not suitable for liquidity pools with both stable tokens exchanged. The exchange of stable tokens needs to minimize the price deviation as much as possible, it will not help attract exchange users by accumulating price deviation and then arbitraging.

FreeSwap is also not applicable to exchange deflationary tokens. As intenal exchange will be conducted between two sub-pools for arbitrage, and the transfer of deflationary token will cause token deflation, which is not beneficial to the sub-pool swapping in deflationary token. Moreover, arbitrage will increase the transfer times of deflationary token, finally accelerate token deflation rate, this effect violates the original intention of deflationary token.

## 4 FreeSwap protocol conclusion

Based on the "constant-product invariant" formula, FreeSwap creates two one-way exchange sub-pools for each pair of tokens. One-way exchange will cause the token prices of two sub-pools to slide in opposite direction. When the the price slippage reaches the preset threshold, the two-pools will exchange with each other internally and automatically for arbitrage. This arbitrage could provide revenue for the liquidity providers, and also smooth the price deviation of two sub-pools.

The arbitrage mechanism of the FreeSwap protocol can maximize the profit for the two sub-pools at the same time, which ensures fairness and achieves a win-win situation. Calculations show that if arbitrage is performed when the price deviation of the sub-pools reaches 1%, FreeSwap's arbitrage mechanism can achieve the profit equivalent to charge 2.488 ‰ of exchange fees while charging no fees at all.

By implementing a completely free decentralized exchange protocol，it is expectable that FreeSwap can help attract more and more users to join in decentralized exchange and take part in more and more Defi activities. 

## References

1、Vitalik Buterin, **Let's run on-chain decentralized exchanges the way we run prediction markets**, Dec. 2016. URL: [https://www.reddit.com/r/ethereum/comments/55m04x/lets_run_onchain_decentralized_exchanges_the_way](https://www.reddit.com/r/ethereum/comments/55m04x/lets_run_onchain_decentralized_exchanges_the_way).

2、 Hayden Adams, **Uniswap Whitepaper**. URL: [https://hackmd.io/C-DvwDSfSxuh-Gd4WKE_ig](https://hackmd.io/C-DvwDSfSxuh-Gd4WKE_ig).

3、 Hayden Adams et al, **Uniswap V2 Core**. URL: [https://uniswap.org/whitepaper.pdf](https://uniswap.org/whitepaper.pdf).

4、 Vitalik Buterin, **The x*y=k market maker model**. URL: [https://ethresear.ch/t/improving-front-running-resistance-of-x-y-k-market-makers](https://ethresear.ch/t/improving-front-running-resistance-of-x-y-k-market-makers).

5、Pintail, **Understanding Uniswap Returns**. URL: [https://medium.com/@pintail/understanding-uniswap-returns-cc593f3499ef](https://medium.com/@pintail/understanding-uniswap-returns-cc593f3499ef).

6、Pintail, **Uniswap: A Good Deal For Liquidity Providers?**. URL: [https://medium.com/@pintail/uniswap-a-good-deal-for-liquidity-providers-104c0b6816f2](https://medium.com/@pintail/uniswap-a-good-deal-for-liquidity-providers-104c0b6816f2).