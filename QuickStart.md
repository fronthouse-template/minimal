Here’s your cleaned-up and nicely formatted markdown:

---

## 🚀 Clone the Template Repository

```bash
git clone https://github.com/fronthouse-template/minimal.git
```

---

## ✏️ Edit & Commit Your Configuration

After editing `config.json`, stage and commit your changes:

```bash
git add .
git commit -m "Add reverse proxy configuration"
```

---

## 📤 Push Your Configuration to FrontHouse

Replace `[YOUR-TENANT-NAME]` with your tenant ID:

```bash
git push https://git.fronthouse.ai/[YOUR-TENANT-NAME]
```

### 🔍 Example

```bash
git push https://git.fronthouse.ai/my-tenant
```

---

## 🪪 Note: Save Your Token!

After your first push, you’ll receive a token like this:

```text
remote: ----------------------------------------------------------------------------------
remote: --> [Token] Generated token for 'd3': hed62oia8u01ae2dmr21r4unsfu8bk42
remote: --> Please record this for future access to this tenant
remote: --> Append onto the tenant name. eg.
remote: --> git push https://git.fronthouse.ai/my-tenant:hed62oia8u01ae2dmr21r4unsfu8bk42
remote: ----------------------------------------------------------------------------------
```

---

## 🔁 Use the Token for Future Pushes

```bash
git push https://git.fronthouse.ai/[YOUR-TENANT-NAME]:[TOKEN]
```

### Example

```bash
git push https://git.fronthouse.ai/my-tenant:hed62oia8u01ae2dmr21r4unsfu8bk42
```

---

## 🎯 Optional: Add a Named Remote

To make things easier, add a remote called `fronthouse`:

```bash
git remote add fronthouse https://git.fronthouse.ai/my-tenant:hed62oia8u01ae2dmr21r4unsfu8bk42
```

Then for future pushes, just use:

```bash
git push fronthouse
```

---

Let me know if you want this embedded in a README or styled for a docs site!