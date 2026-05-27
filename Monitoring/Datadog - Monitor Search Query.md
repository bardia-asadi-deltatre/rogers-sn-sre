# Datadog - Monitor Search Query

Query to find all Sportsnet+ monitors in Datadog:

```
(tag:"customer:rogers" AND (tag:"app_owner:VI" OR tag:"app_owner:DivaBO")) OR tag:"customer:rogers-diva" OR tag:"project:rogers-diva" OR tag:"project:axis-rogers-diva" OR tag:"axis.project_code:rogers-diva" OR tag:"project:axis-rogers"
```

Tags are inconsistent across teams — this query is the result of significant investigation. See `current-state.md` for full context.
