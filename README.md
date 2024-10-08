RWA-Rarity Wine Asset
The RWA project aspires to revolutionize the rare wine market by establishing a secure, transparent, and efficient digital marketplace.
// Solana Blockchain Smart Contract Template for NFT Marketplace and Tokenized Wine Assets

use anchor_lang::prelude::*;
use anchor_spl::token::{self, Mint, Token, TokenAccount};

declare_id!("YOUR_PROGRAM_ID");

#[program]
pub mod wine_nft_platform {
    use super::*;

  // Function to initialize the platform and mint an NFT for a wine asset
    pub fn initialize_nft(
        ctx: Context<InitializeNFT>, 
        metadata_uri: String, 
        price: u64,
        royalty: u64,
    ) -> Result<()> {
        let nft_data = &mut ctx.accounts.nft_data;
        nft_data.metadata_uri = metadata_uri;
        nft_data.price = price;
        nft_data.royalty = royalty;
        Ok(())
    }

  // Function to facilitate the sale of the NFT (Wine Asset)
    pub fn sell_nft(ctx: Context<SellNFT>, sale_price: u64) -> Result<()> {
        let seller = &ctx.accounts.seller;
        let buyer = &ctx.accounts.buyer;
        let nft = &ctx.accounts.nft;

  // Transfer NFT from seller to buyer
        token::transfer(
            ctx.accounts.into_transfer_context(),
            nft.amount,
        )?;
        
  // Distribute royalty to the original creator
        let royalty_payment = sale_price * nft.royalty / 100;
        token::transfer(
            ctx.accounts.into_royalty_context(),
            royalty_payment,
        )?;

  // Transfer remaining sale amount to seller
        let seller_payment = sale_price - royalty_payment;
        token::transfer(
            ctx.accounts.into_payment_context(),
            seller_payment,
        )?;

  Ok(())
    }

  // Function to manage fractional ownership
    pub fn fractionalize_ownership(
        ctx: Context<FractionalizeOwnership>,
        fractions: u64,
    ) -> Result<()> {
        let nft_data = &mut ctx.accounts.nft_data;
        nft_data.fractionalized = true;
        nft_data.fractions = fractions;
        Ok(())
    }
}

#[derive(Accounts)]
pub struct InitializeNFT<'info> {
    #[account(init, payer = user, space = 8 + 64)]
    pub nft_data: ProgramAccount<'info, NFTData>,
    #[account(mut)]
    pub user: Signer<'info>,
    pub system_program: Program<'info, System>,
    pub rent: Sysvar<'info, Rent>,
    pub token_program: Program<'info, Token>,
    pub mint: Account<'info, Mint>,
}

#[derive(Accounts)]
pub struct SellNFT<'info> {
    #[account(mut)]
    pub seller: Signer<'info>,
    #[account(mut)]
    pub buyer: Signer<'info>,
    #[account(mut)]
    pub nft: Account<'info, TokenAccount>,
    #[account(mut)]
    pub nft_data: ProgramAccount<'info, NFTData>,
}

#[derive(Accounts)]
pub struct FractionalizeOwnership<'info> {
    #[account(mut)]
    pub nft_data: ProgramAccount<'info, NFTData>,
    pub owner: Signer<'info>,
}

#[account]
pub struct NFTData {
    pub metadata_uri: String,
    pub price: u64,
    pub royalty: u64,
    pub fractionalized: bool,
    pub fractions: u64,
}
