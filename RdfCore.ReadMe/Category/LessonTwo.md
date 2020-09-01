# Rdf.Data��ʹ��
- [1.��������](#1��������)
- [2.��ȡʵ�弯��](#2��ȡʵ�弯��)
- [3.��ҳ��ȡʵ����б�](#3��ҳ��ȡʵ����б�)
- [4.��ȡ����ʵ��](#4��ȡ����ʵ��)
- [5.���²���](#5���²���)
- [6.����ɾ������](#6����ɾ������)
- [7.�Զ���Sql��ѯ](#7�Զ���Sql��ѯ)

## 1.��������
```
public async Task<int> CreateRole(CreateRoleInput input, string tenantId, string userId)
{
    if (string.IsNullOrWhiteSpace(input.RoleName))
        throw new CustomException("��ɫ���Ʋ���Ϊ�գ�");

    if (await IsExistAsync(p => p.RoleName == input.RoleName))
        throw new CustomException($@"{input.RoleName}��ɫ�����Ѿ����ڣ�������ʹ��һ���½�ɫ���ƣ�");

    var entity = new Role
    {
        RoleId = GuidHelper.GenerateNewId(),
        RoleName = input.RoleName,
        RoleType = input.RoleType,
        RoleDesc = input.RoleDesc,
        IsStatic = input.IsStatic,
        IsDefault = input.IsDefault,
        DispOrder = input.DispOrder,
        TenantId = tenantId,
        IsDeleted = 0,
        CreatorUserId = userId,
        CreationTime = System.DateTime.Now
    };

    return await InsertAsync(entity);
}
```
## 2.��ȡʵ�弯��
```
public async Task<List<GetDetailRoleOutput>> GetAllRole(string tenantId, string userId)
{
    var query = GetAll().Select(o => new GetDetailRoleOutput
    {
        RoleId = o.RoleId,
        RoleName = o.RoleName,
        RoleType = o.RoleType,
        RoleDesc = o.RoleDesc,
        IsStatic = o.IsStatic,
        IsDefault = o.IsDefault,
        DispOrder = o.DispOrder
    }).OrderBy(p => p.DispOrder);

    return await query.ToListAsync();
}
```

## 3.��ҳ��ȡʵ����б�
```
public async Task<PageOutput> GetPageRole(GetPageRoleInput input, string tenantId, string userId)
{
    if (string.IsNullOrEmpty(input.OrderBy))
        input.OrderBy = "DISP_ORDER";

    Expression<Func<Role, bool>> where = (it => 1 == 1);

    if (!string.IsNullOrEmpty(input.Keyword))
    {
        Expression<Func<Role, bool>> exp1 = (p => p.RoleName.Contains(input.Keyword));
        where = ExpressionHelp.And(where, exp1);
    }

    return await GetPageAsync(where, input.OrderBy, input.PageIndex, input.PageSize);
}
```

## 4.��ȡ����ʵ��
```
public async Task<GetDetailRoleOutput> GetDetailRole(GetDetailInput<string> input, string tenantId, string userId)
{
    var o = await GetByIdAsync(input.Id);

    var output = new GetDetailRoleOutput
    {
        RoleId = o.RoleId,
        RoleName = o.RoleName,
        RoleType = o.RoleType,
        RoleDesc = o.RoleDesc,
        IsStatic = o.IsStatic,
        IsDefault = o.IsDefault,
        DispOrder = o.DispOrder,
        TenantId = o.TenantId,
        IsDeleted = o.IsDeleted,
        DeleterUserId = o.DeleterUserId,
        DeletionTime = o.DeletionTime,
        LastModifierUserId = o.LastModifierUserId,
        LastModificationTime = o.LastModificationTime,
        CreatorUserId = o.CreatorUserId,
        CreationTime = o.CreationTime
    };
    return output;
}
```

## 5.���²���
```
public async Task<int> UpdateRole(UpdateRoleInput input, string tenantId, string userId)
{
    if (await IsExistAsync(p => p.RoleName == input.RoleName && p.RoleId != input.RoleId))
        throw new CustomException($@"{input.RoleName}��ɫ�����Ѵ����������ظ�������");

    var entity = new Role
    {
        RoleId = input.RoleId,
        RoleName = input.RoleName,
        RoleType = input.RoleType,
        RoleDesc = input.RoleDesc,
        IsStatic = input.IsStatic,
        IsDefault = input.IsDefault,
        DispOrder = input.DispOrder,
        TenantId = tenantId,
        IsDeleted = input.IsDeleted == null ? 0 : input.IsDeleted,
        LastModifierUserId = userId,
        LastModificationTime = System.DateTime.Now
    };

    return await UpdateAsync(entity);
}
```

## 6.����ɾ������
```
public async Task<int> BatchDeleteRole(BatchDeleteInput<string> input, string tenantId, string userId)
{
    return await DeleteAsync(p => input.Ids.Contains(p.RoleId));
}
```

## 7.�Զ���Sql��ѯ
```
public async Task<IEnumerable<GetDetailRoleOutput>> GetAllRole2(string tenantId, string userId)
{
    #region sqlQuery
    var sqlQuery = $@"SELECT T1.ROLE_ID RoleId
            ,T1.ROLE_NAME RoleName
            ,T1.ROLE_TYPE RoleType
            ,T1.ROLE_DESC RoleDesc
            ,T1.IS_STATIC IsStatic
            ,T1.IS_DEFAULT IsDefault
            ,T1.DISP_ORDER DispOrder
        FROM ACT_ROLE T1
        ORDER BY T1.DISP_ORDER";
    #endregion

    return await SqlQueryAsync<GetDetailRoleOutput>(sqlQuery);
}
```