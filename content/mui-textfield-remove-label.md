---
title: "【MUI】How to remove label in TextField"
date: 2021-10-21
tags: ["React", "MUI"]
---

```tsx:sample.tsx
<TextField
  sx={{
    '& legend': { display: 'none' },
    '& fieldset': { top: 0 },
  }}
/>
```

## Result
![](/images/6.gif)
