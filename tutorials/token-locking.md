# 🔒 Token Locking

MNGO tokens can be locked for up to 5 years and receive increased voting weight in the DAO up to 2x. Voting weight is calculated linearly based on locking period. Locked tokens cannot be withdrawn until lockup expires.\
\
\*If you have tokens deposited into old governance, you will need to withdraw and redeposit to vote.\
\
First, navigate to the MangoDAO: [https://dao-beta.mango.markets/dao/MNGO](https://dao-beta.mango.markets/dao/MNGO), connect your wallet, and deposit MNGO.

The "Your Tokens" panel will populate with your deposited MNGO amount.

![](<../.gitbook/assets/Screen Shot 2022-03-02 at 4.50.04 PM.png>)\
\
MNGO Deposited is the amount of unlocked MNGO in the governance. MNGO Locked represents the amount of MNGO that is locked unable to withdraw (0 in the above case).

Next, click "Manage" to bring up the Your Tokens Overview page. Select "Lock Tokens" button to begin the lockup process.

![](<../.gitbook/assets/Screen Shot 2022-03-02 at 4.53.17 PM.png>)

You are then presented with the Lockup Type Options of: Cliff, Constant, and Vested. The differences between these types are below.\
\
**Cliff**: Tokens are locked for a fixed duration and are released in full at the end of it. Vote weight declines linearly until release. Example: You lock 10.000 tokens for two years. They are then unavailable for the next two years. After this time, you can withdraw them again.

**Constant**: Tokens are locked indefinitely. At any time you can start the unlock process which lasts for the initially chosen lockup duration. Vote weight stays constant until you start the unlock process, then it declines linearly until release. Example: You lock 10.000 tokens with a lockup duration of one year. After two years you decide to start the unlocking process. Another year after that, you can withdraw the tokens.

**Vested**: Tokens are locked for a fixed duration and released over time at a rate of _locked amount/number of periods_ per vesting period. Vote weight declines linearly and with each vesting until release. Example: You lock 12.000 tokens for one year with monthly vesting. Every month 1.000 tokens unlock. After the year, all tokens have unlocked.

Enter the desired Lockup Type, Amount to Lock, and Duration. The Vote Weight Multiplier shows how much bonus voting power you will receive on your tokens.

![](<../.gitbook/assets/Screen Shot 2022-03-02 at 5.00.43 PM.png>)\


Once the form is complete, click "Lock Tokens." You will be prompted with the a confirmation, select Confirm to finalise the locking process.

Your Token Overview will repopulate with an overview of your locking schedule and total votes in the DAO.

You can withdraw your tokens once the locking period expires.
