--------------------------Nashville Housing Data Cleaning--------------------------------------

--Nashville Housing Table

select * from Nashville_Housing

--------------------------------------------------------------------------------------------

--Standardize date format

select SaleDate from Nashville_Housing


alter table Nashville_Housing
add SaleDateConverted date

update Nashville_Housing
set SaleDateConverted = CONVERT(date,SaleDate)

----------------------------------------------------------------------------------------


--Populate property address data

select a.ParcelID,a.PropertyAddress,b.ParcelID,b.PropertyAddress,
ISNULL(a.PropertyAddress,b.PropertyAddress) as FilledPropertyAddress
from Nashville_Housing a 
join Nashville_Housing b
	 on a.ParcelID = b.ParcelID
	 and a.[UniqueID ] <> b.[UniqueID ]
where a.PropertyAddress is null 

update a
set PropertyAddress = ISNULL(a.PropertyAddress,b.PropertyAddress)
from Nashville_Housing a 
join Nashville_Housing b
	 on a.ParcelID = b.ParcelID
	 and a.[UniqueID ] <> b.[UniqueID ]
where a.PropertyAddress is null 

----------------------------------------------------------------------------------------------

--Breaking out address into individual colunms (Address, City, State)

--[To perform operation i used Substring function as my fist method to split the PROPERTY ADDRESS]
select PropertyAddress,SUBSTRING(PropertyAddress,1,CHARINDEX(',',PropertyAddress)-1) AS Address,
SUBSTRING(PropertyAddress,CHARINDEX(',',PropertyAddress) +1,LEN(PropertyAddress)) as city
from Nashville_Housing

alter table Nashville_Housing
add PropertySplitAddress varchar(100)

update Nashville_Housing
set PropertySplitAddress = SUBSTRING(PropertyAddress,1,CHARINDEX(',',PropertyAddress)-1)

alter table Nashville_Housing
add PropertySplitCity varchar(100)

update Nashville_Housing
set PropertySplitCity = SUBSTRING(PropertyAddress,CHARINDEX(',',PropertyAddress) +1,LEN(PropertyAddress))

--[Now I'm splitting OwnerAddress using Parsename function]

select OwnerAddress, 
PARSENAME(REPLACE(OwnerAddress,',','.'),3) as Address,
PARSENAME(REPLACE(OwnerAddress,',','.'),2) as City,
PARSENAME(REPLACE(OwnerAddress,',','.'),1)as State
from Nashville_Housing


--------------------------------------------------------------------------------------------------------------------

--Change Y and N to Yes and No in 'SoldAsVacant' field

select distinct SoldAsVacant,count(SoldAsVacant)
from Nashville_Housing
group by SoldAsVacant

select SoldAsVacant,
 CASE 
	when SoldAsVacant = 'N' then 'No'
	when SoldAsVacant = 'Y' then 'Yes'
	else SoldAsVacant
	end
from Nashville_Housing


update  Nashville_Housing
set SoldAsVacant =  CASE 
	when SoldAsVacant = 'N' then 'No'
	when SoldAsVacant = 'Y' then 'Yes'
	else SoldAsVacant
	end

--------------------------------------------------------------------------------------------

--Remove Duplicate

--[Using windows function]

with RowNumCTE as(
select *,
ROW_NUMBER() over (PARTITION BY ParcelID,PropertyAddress,SalePrice,SaleDate,
LegalReference order by UniqueID) row_num
from Nashville_Housing
)

select * from RowNumCTE
where row_num > 1
--order by PropertyAddress

-----------------------------------------------------------------------------------------------

--Deleting unused columns

alter table Nashville_Housing
drop column PropertyAddress,SaleDate,OwnerAddress

