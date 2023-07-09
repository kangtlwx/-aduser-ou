以下为powershell脚本内容，csv文件请参考以下格式，需要包含以下列：
employeeID,description,sn,givenName,displayname,name,userprincipalname,sAMAccountName,EmailAddress,SipAddress,mailNickname,mail,company,department,title
# 导入 CSV 文件
# csv：employeeID,description,sn,givenName,displayname,name,userprincipalname,sAMAccountName,EmailAddress,SipAddress,mailNickname,mail,company,department,title
$departments = Import-Csv -Path "C:\PS\usersinfo-new.csv" | Sort-Object -Property Department

# 循环遍历每个部门
foreach ($department in $departments) {
    # 检查 OU 是否已存在，如果不存在则创建 OU
    $ouPath = "OU=" + $department.'Department' + ",OU=testou,DC=abc,DC=com"
    if (!(Get-ADOrganizationalUnit -Filter "Name -eq '$($department.'Department')'")) {
        New-ADOrganizationalUnit -Name $department.'Department' -Path "OU=testou,DC=abc,DC=com"
    }
    
    # 获取该部门的所有用户
    $users = Get-ADUser -Filter "Department -eq '$($department.'Department')'"
    
    # 循环遍历部门中的每个用户，并将其移动到相应的 OU 中
    foreach ($user in $users) {
        Move-ADObject -Identity $user.DistinguishedName -TargetPath $ouPath
    }
}
