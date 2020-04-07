##### Gitlab host cli  
Go to Ruby on Rails console
```bash
gitlab-rails console -e production
```
Console load is slowly
```bash
user = User.where(id: 1).first
user.password = 'secret_pass'
user.password_confirmation = 'secret_pass'
user.save!
```
##### Ref  
https://docs.gitlab.com/ee/security/reset_root_password.html