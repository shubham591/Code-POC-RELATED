WITH DuplicateStatusCodes AS (
    SELECT 
        status_code
    FROM 
        audit_status_table
    WHERE 
        status_code IN (5, 7)  -- Assuming 5 = draft, 7 = rework
    GROUP BY status_code
    HAVING COUNT(*) > 1  -- Only status codes with more than one entry
),
CTE AS (
    SELECT 
        status_id, 
        status_code, 
        ROW_NUMBER() OVER (PARTITION BY status_code ORDER BY status_date) AS rn
    FROM 
        audit_status_table
    WHERE 
        status_code IN (SELECT status_code FROM DuplicateStatusCodes)
)
DELETE FROM audit_status_table
WHERE status_id IN (
    SELECT status_id 
    FROM CTE 
    WHERE rn > 1  -- Delete only duplicate entries, keep the first one
);
