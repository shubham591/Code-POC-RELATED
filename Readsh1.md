WITH DuplicateStatusCodes AS (
    SELECT 
        status_code
    FROM 
        audit_status_table
    WHERE 
        status_code IN (5, 7)
    GROUP BY status_code
    HAVING COUNT(*) > 1
),
CTE AS (
    SELECT 
        status_id, 
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
    WHERE rn > 1
);
