### Blind Auction

Bu bölümde, Ethereum üzerinde tamamen gizli bir müzayede sözleşmesi oluşturmanın ne kadar kolay olduğunu göstereceğiz. Herkesin yapılan teklifleri görebileceği bir açık artırma ile başlayacağız. Daha sonra bu sözleşmeyi, teklif süresi bitene kadar gerçek teklifi görmenin mümkün olmadığı gizli müzayedeye dönüştüreceğiz.

### Basit Herkesin Katılabileceği Açık Artırma

Aşağıdaki basit müzayede sözleşmesinin genel fikri, herkesin tekliflerini bir teklif verme süresi boyunca gönderebilmesidir. Teklifler, teklif verenleri tekliflerine bağlamak için zaten para(Ether) göndermeyi içeriyor. En yüksek teklif yükseltilirse, bir önceki en yüksek teklifi veren parasını geri alır. Teklif süresinin bitiminden sonra, lehtarın parasını alması için sözleşmenin manuel olarak çağrılması gerekir - sözleşmeler kendiliğinden etkinleştirilemez.

// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.4;
contract SimpleAuction {
    // Parameters of the auction. Times are either
    // absolute unix timestamps (seconds since 1970-01-01)
    // or time periods in seconds.
    address payable public beneficiary;
    uint public auctionEndTime;

    // Current state of the auction.
    address public highestBidder;
    uint public highestBid;

    // Allowed withdrawals of previous bids
    mapping(address => uint) pendingReturns;

    // Set to true at the end, disallows any change.
    // By default initialized to `false`.
    bool ended;

    // Events that will be emitted on changes.
    event HighestBidIncreased(address bidder, uint amount);
    event AuctionEnded(address winner, uint amount);

    // Errors that describe failures.

    // The triple-slash comments are so-called natspec
    // comments. They will be shown when the user
    // is asked to confirm a transaction or
    // when an error is displayed.

    /// The auction has already ended.
    error AuctionAlreadyEnded();
    /// There is already a higher or equal bid.
    error BidNotHighEnough(uint highestBid);
    /// The auction has not ended yet.
    error AuctionNotYetEnded();
    /// The function auctionEnd has already been called.
    error AuctionEndAlreadyCalled();

    /// Create a simple auction with `biddingTime`
    /// seconds bidding time on behalf of the
    /// beneficiary address `beneficiaryAddress`.
    constructor(
        uint biddingTime,
        address payable beneficiaryAddress
    ) {
        beneficiary = beneficiaryAddress;
        auctionEndTime = block.timestamp + biddingTime;
    }

    /// Bid on the auction with the value sent
    /// together with this transaction.
    /// The value will only be refunded if the
    /// auction is not won.
    function bid() external payable {
        // No arguments are necessary, all
        // information is already part of
        // the transaction. The keyword payable
        // is required for the function to
        // be able to receive Ether.

        // Revert the call if the bidding
        // period is over.
        if (block.timestamp > auctionEndTime)
            revert AuctionAlreadyEnded();

        // If the bid is not higher, send the
        // money back (the revert statement
        // will revert all changes in this
        // function execution including
        // it having received the money).
        if (msg.value <= highestBid)
            revert BidNotHighEnough(highestBid);

        if (highestBid != 0) {
            // Sending back the money by simply using
            // highestBidder.send(highestBid) is a security risk
            // because it could execute an untrusted contract.
            // It is always safer to let the recipients
            // withdraw their money themselves.
            pendingReturns[highestBidder] += highestBid;
        }
        highestBidder = msg.sender;
        highestBid = msg.value;
        emit HighestBidIncreased(msg.sender, msg.value);
    }

    /// Withdraw a bid that was overbid.
    function withdraw() external returns (bool) {
        uint amount = pendingReturns[msg.sender];
        if (amount > 0) {
            // It is important to set this to zero because the recipient
            // can call this function again as part of the receiving call
            // before `send` returns.
            pendingReturns[msg.sender] = 0;

            // msg.sender is not of type `address payable` and must be
            // explicitly converted using `payable(msg.sender)` in order
            // use the member function `send()`.
            if (!payable(msg.sender).send(amount)) {
                // No need to call throw here, just reset the amount owing
                pendingReturns[msg.sender] = amount;
                return false;
            }
        }
        return true;
    }

    /// End the auction and send the highest bid
    /// to the beneficiary.
    function auctionEnd() external {
        // It is a good guideline to structure functions that interact
        // with other contracts (i.e. they call functions or send Ether)
        // into three phases:
        // 1. checking conditions
        // 2. performing actions (potentially changing conditions)
        // 3. interacting with other contracts
        // If these phases are mixed up, the other contract could call
        // back into the current contract and modify the state or cause
        // effects (ether payout) to be performed multiple times.
        // If functions called internally include interaction with external
        // contracts, they also have to be considered interaction with
        // external contracts.

        // 1. Conditions
        if (block.timestamp < auctionEndTime)
            revert AuctionNotYetEnded();
        if (ended)
            revert AuctionEndAlreadyCalled();

        // 2. Effects
        ended = true;
        emit AuctionEnded(highestBidder, highestBid);

        // 3. Interaction
        beneficiary.transfer(highestBid);
    }
}

### Gizli Müzayede(Blind Auction)

Önceki herkesin katılabileceği açık artırma, gizli açık artırmaya dönüştürülebilir. Gizli müzayedenin avantajı, teklif verme süresinin sonuna doğru herhangi bir zaman baskısı olmamasıdır. Şeffaf bir bilgi işlem platformunda gizli bir müzayede oluşturmak bir çelişki gibi görünebilir, ancak kriptografi buna mahal vermeyecektir.

Teklif verme süresi boyunca, teklif sahibi teklifini gerçekte göndermez, yalnızca şifreli(hashed) bir sürümünü gönderir. Şu anda hash değerleri eşit olan iki (yeterince uzun) değer bulmak pratik olarak imkansız kabul edildiğinden, teklif sahibi teklifi bu şekilde taahhüt eder. Teklif verme süresinin bitiminden sonra, teklif sahipleri tekliflerini açıklamak zorundadır: Değerlerini şifresiz olarak gönderirler ve sözleşme, hash değerinin teklif döneminde sağlanan ile aynı olup olmadığını kontrol eder.

Bir diğer zorluk, müzayedeyi aynı anda hem **bağlayıcı** hem de **gizli** hale getirmektir: Teklif sahibinin müzayedeyi kazandıktan sonra parayı göndermemesini engellemenin tek yolu, teklifle birlikte göndermelerini sağlamaktır. Ethereum'da değer transferleri gizli edilemediği için değeri herkes görebilir.

Aşağıdaki sözleşme, en yüksek tekliften daha büyük herhangi bir değeri kabul ederek bu sorunu çözmektedir. Bu, elbette yalnızca açıklama aşamasında kontrol edilebildiğinden, bazı teklifler kasıtlı olarak geçersiz olabilir(hatta yüksek değer transferleriyle geçersiz teklifler vermek için açık bir işaret sağlar): Bu sayede ihaleye katılanlar birkaç tane çok yüksek veya çok alçak geçersiz teklifle rekabeti kızıştırabilirler.