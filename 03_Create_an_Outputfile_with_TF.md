## **📌 How to Create an Output File with Terraform**
When running `terraform apply` or `terraform plan`, you may want to save the output for **logging, debugging, or automation**. Terraform provides multiple ways to capture output.

---

### **1️⃣ Redirect Output to a File**
If you want to save Terraform's output while still displaying it in the terminal, use `tee`:
```sh
terraform apply | tee terraform_output.md
```
✅ **This will:**
- Show the output in the terminal.
- Save the same output to **`terraform_output.md`**.

If you **don't** want the output to be displayed in the terminal (**only save it**), use:
```sh
terraform apply > terraform_output.md 2>&1
```
✅ This redirects **both standard output (`stdout`) and errors (`stderr`)** to a file.

---

### **2️⃣ Save Terraform Plan Output**
If you want to save the **planned changes** before applying them:
```sh
terraform plan | tee terraform_plan.md
```
or
```sh
terraform plan > terraform_plan.md 2>&1
```
✅ This helps review what Terraform **will change** before running `terraform apply`.

---

### **3️⃣ Append Output to an Existing Log File**
To continuously log multiple `terraform apply` runs into a single file:
```sh
terraform apply | tee -a terraform.log
```
✅ The `-a` flag ensures **new output is added** instead of overwriting the file.

---

### **4️⃣ Save JSON Output for Automation**
If you need a **machine-readable JSON output**, use:
```sh
terraform plan -out=tfplan
terraform show -json tfplan > terraform_output.json
```
✅ This stores Terraform’s execution plan in **JSON format**, useful for scripts or automation.

---

## **📌 Summary of Commands**
| Command | Purpose |
|---------|---------|
| `terraform apply | tee terraform_output.md` | Save output **and** display in the terminal. |
| `terraform apply > terraform_output.md 2>&1` | Save output **without** displaying it. |
| `terraform plan | tee terraform_plan.md` | Save the **plan output** before applying changes. |
| `terraform apply | tee -a terraform.log` | Append `terraform apply` output to an existing log file. |
| `terraform plan -out=tfplan && terraform show -json tfplan > terraform_output.json` | Save Terraform output in **JSON format** for automation. |

---

### **📌 Best Practices**
✅ **Use `tee`** when you want to **see output while saving it**.  
✅ **Redirect (`>`) output** when running Terraform in **CI/CD pipelines**.  
✅ **Use JSON output (`terraform show -json`)** for automation scripts.  
✅ **Always review `terraform plan` before applying changes**.

Now, you can efficiently track Terraform changes and maintain logs! 🚀

---