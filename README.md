# Symplique Solutions Azure Engineer Assessment

# Strategy:
- Migrate billing records older than 3 months to Azure Blob storage with Cool Tier (as response is expected in seconds. Use Hot tier if milli second latency is needed.
The API contract remains the same.

- Internal fallback logic similar to "try except block" can be implemented which will first search records in Cosmos DB and if the record is not found it looks into the Blob storage.

# Fallback Logic:
<p><pre>
try:
  #Azure Cosmos DB access
except:
  #Blob storage access
</pre></p>

# Architecture Diagarm:
<img width="539" alt="image" src="https://github.com/user-attachments/assets/28be130e-9f66-4595-b94c-dbe1f9459c80" />

# Problems might be faced:
- To maintain data integrity, we can
  1. Write to Blob
  2. Confirm Blob Integrity
  3. Then delete from Cosmos DB
- Security. No public access, signed URLs and strict access control      
# ChatGPT Prompts:
- Me: Can you give tell me the risks involving data loss while migrating from Cosmos DB to Blob and ways to mitigate them?
- GPT: Sure — one of the main risks is data loss during archival, especially if the system deletes from Cosmos DB before confirming a successful Blob write. I’d mitigate that by using a two-step archival flow — first upload, verify integrity, then delete.
Another concern is latency from Blob Storage; I'd log these events, monitor usage patterns, and potentially cache frequently-accessed older records.
Security-wise, I’d enforce private access and use managed identities for accessing Blob securely. Finally, I’d ensure proper observability — adding metrics to track how often fallbacks occur, when they fail, and alert thresholds for anomalies
- Me: What should I worry about when using Cosmos DB + Blob fallback?
- GPT: You should watch for:
Data inconsistency during archival: if you delete from Cosmos before confirming the Blob write, records may vanish.
Blob read latency: Cold tier storage may respond in 1–3 seconds, which could feel sluggish if users expect sub-second reads.
Format mismatch: A record from Blob might differ in structure (e.g., missing metadata or extra fields), so your API must normalize this.
Blob unavailability: If Blob Storage has transient failures or permission issues (e.g., expired SAS tokens), your fallback will break.
Fallback overload: If 30% of queries go to Blob, your storage costs and latency grow — especially if records are not cached or rehydrated
