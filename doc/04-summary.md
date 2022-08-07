## 📖 Summary

- Created 3 `deployments` and `services` to represent each microservice 
  - 1️⃣ `redis`
  - 2️⃣ `api`
  - 3️⃣ `pinger` 
  - 4️⃣ `poller`

- Deployed to our existing nodejs-demo k3d cluster

- validated all servies are running via: 
  - port-forward to `pod`
  - port-forward to `service`
  - review container `logs`
  - run `redis-cli` aginst the redis service
  - customize command + args
  - add environment variables
  - set resource limits

---

🆙 next - [HELM](05-00-helm.md) 