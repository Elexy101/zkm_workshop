# zkm_workshop
A use-case i am using to build a CoinFlip game on ZKM 

The **CoinFlip game** is a simple betting game where a player flips a virtual coin, and the outcome (heads or tails) determines whether they win or lose. With Zero-Knowledge Machine (ZKM) integration, we ensure that both the players and the contract operator can verify the fairness of the game without revealing sensitive details like random numbers or bets.

**Components Involved**

1. ZKM (Zero-Knowledge Machine): To ensure privacy and fairness through zero-knowledge proofs (ZKPs).

2. Smart Contract: To manage game logic, betting, and payouts.

3. Players: Users participating in the game by placing bets.

4. Verifier: The contract or external verifier ensures that the result was fair and no manipulation occurred.

** Use Case Flow **
**1. Bet Placement by the Player**
Input: Player chooses heads (1) or tails (0) and specifies the amount they want to bet.
Smart Contract Logic (Rust): The bet and the player's choice are hashed and committed to the blockchain without revealing the actual choice (heads or tails).

**2. Randomness Generation**
ZKM (Go): The contract calls the ZKM via Go code to generate a secure random number using ZKP. The random number determines the outcome of the coin flip.
ZKP: The Go module generates a zero-knowledge proof showing that the random number was generated fairly.

**3. Commitment and Hashing**
The contract hashes the random number and commits it to the blockchain. This ensures that no one can change the result after it is generated.

**4. Result Reveal**
The smart contract reveals the result (heads or tails) to the player and verifies the fairness of the result using the zero-knowledge proof.

**5. Payout**
If the player guessed correctly, they are rewarded according to the bet amount. If they lose, the contract keeps the bet.


## SMART CONTRACT CODE 

use zk_proofs::{generate_proof, verify_proof}; // Placeholder for Rust ZKP library (or bindings to Go library)
use sha3::{Digest, Sha3_256}; // Secure hashing for commitments

// Struct for player bet information
struct Bet {
    player_address: String,
    bet_choice: bool,  // true for heads, false for tails
    bet_amount: u64,   // Amount staked by the player
    commitment: String // Hash of the bet choice and player address
}

// Function for placing a bet
fn place_bet(player_address: &str, bet_choice: bool, bet_amount: u64) -> Bet {
    // Step 1: Create a hash commitment of the player's choice (heads or tails) and their address
    let mut hasher = Sha3_256::new();
    hasher.update(player_address.as_bytes());
    hasher.update(&[bet_choice as u8]);  // Convert bool to byte
    let commitment = hasher.finalize();

    // Step 2: Create a new Bet instance and return
    Bet {
        player_address: player_address.to_string(),
        bet_choice,
        bet_amount,
        commitment: format!("{:x}", commitment),
    }
}

// Function to trigger the coin flip and check the result
fn flip_coin_and_check_result(bet: &Bet) -> Result<bool, &'static str> {
    // Step 3: Call the Go-based ZKM to generate randomness and get the ZKP
    let (random_number, proof) = generate_proof(); // Call to Go module via FFI or API
    
    // Step 4: Verify the proof of fairness
    if !verify_proof(&proof) {
        return Err("Failed to verify zero-knowledge proof");
    }

    // Step 5: Compute the flip result based on randomness (0 for tails, 1 for heads)
    let flip_result = random_number % 2 == 1;

    // Step 6: Compare the flip result with the player's bet choice
    if flip_result == bet.bet_choice {
        Ok(true) // Player wins
    } else {
        Ok(false) // Player loses
    }
}

// Function to handle payouts after the result
fn payout(bet: &Bet, is_winner: bool) {
    if is_winner {
        println!("Player {} won {} coins!", bet.player_address, bet.bet_amount * 2);
        // Handle payout logic here
    } else {
        println!("Player {} lost their bet.", bet.player_address);
        // Bet is forfeited to the contract
    }
}
