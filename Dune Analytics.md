-- This gets individual transactions and the number of sub transactions by address.

```sql
-- Count the number of wallets a single transaction sent tokens to:

WITH to_txns AS 
    (
    SELECT "to" as address, 
        count("evt_tx_hash") as sub_txn_count, 
        sum("value"/{{Decimal Check}}) as number_of_tokens, 
        "evt_tx_hash", 
        "evt_block_time", 
        "contract_address" 
    FROM bep20."BEP20_evt_Transfer"
        WHERE "contract_address" = CONCAT('\x', substring('{{Contract Address}}' from 3))::bytea
        GROUP BY 1,4,5,6
        ORDER BY 5 ASC )
    , from_txns AS 
    (
    SELECT "from" as address, 
        count("evt_tx_hash") as sub_txn_count, 
        sum("value"/{{Decimal Check}}) as number_of_tokens, 
        "evt_tx_hash", 
        "evt_block_time", 
        "contract_address" 
    FROM bep20."BEP20_evt_Transfer"
        WHERE "contract_address" = CONCAT('\x', substring('{{Contract Address}}' from 3))::bytea
        GROUP BY 1,4,5,6
        ORDER BY 5 ASC )
    SELECT 
    *
    FROM to_txns
        WHERE sub_txn_count > 1
        UNION ALL 
    SELECT 
    *
    FROM from_txns
        WHERE sub_txn_count > 1
        GROUP BY from_txns.address, 
            from_txns.sub_txn_count, 
            from_txns.number_of_tokens, 
            from_txns.evt_tx_hash, 
            from_txns.evt_block_time, 
            from_txns.contract_address
        ORDER BY sub_txn_count DESC; 


-- Code for daily transfer volume:

SELECT date_trunc('day', "evt_block_time") as day, 
    sum((value/1000000000000000000)/2) as Daily_Volume
    FROM bep20."BEP20_evt_Transfer"
    WHERE "contract_address" = CONCAT('\x', substring('{{Contract Address}}' from 3))::bytea
    GROUP BY 1
    ORDER BY 1;
```
