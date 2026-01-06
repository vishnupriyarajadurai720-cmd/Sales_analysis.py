import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from datetime import datetime, timedelta
import warnings
warnings.filterwarnings('ignore')

# Set style
sns.set_style("whitegrid")
plt.rcParams['figure.figsize'] = (12, 6)

# Generate Realistic Sales Dataset
def generate_sales_data():
    """Generate realistic business sales data for analysis"""
    
    np.random.seed(42)
    
    # Product catalog
    products = [
        {'name': 'Laptop Pro X1', 'category': 'Electronics', 'price': 1200, 'margin': 0.25},
        {'name': 'Wireless Mouse', 'category': 'Electronics', 'price': 45, 'margin': 0.40},
        {'name': 'Office Chair Elite', 'category': 'Furniture', 'price': 350, 'margin': 0.35},
        {'name': 'Standing Desk', 'category': 'Furniture', 'price': 600, 'margin': 0.30},
        {'name': 'Coffee Maker Pro', 'category': 'Appliances', 'price': 180, 'margin': 0.38},
        {'name': 'Blender Deluxe', 'category': 'Appliances', 'price': 120, 'margin': 0.42},
        {'name': 'Smartphone Z5', 'category': 'Electronics', 'price': 800, 'margin': 0.22},
        {'name': 'Tablet Mini', 'category': 'Electronics', 'price': 400, 'margin': 0.28},
        {'name': 'Bookshelf Oak', 'category': 'Furniture', 'price': 280, 'margin': 0.33},
        {'name': 'LED Lamp', 'category': 'Electronics', 'price': 65, 'margin': 0.45}
    ]
    
    regions = ['North', 'South', 'East', 'West', 'Central']
    
    # Generate data for 12 months
    sales_records = []
    start_date = datetime(2024, 1, 1)
    
    for month in range(12):
        current_date = start_date + timedelta(days=30*month)
        
        for product in products:
            for region in regions:
                # Add seasonality and regional variation
                seasonal_factor = 1 + np.sin((month / 12) * 2 * np.pi) * 0.3
                regional_factor = np.random.uniform(0.8, 1.2)
                
                # Generate quantity sold
                base_quantity = np.random.randint(10, 50)
                quantity = int(base_quantity * seasonal_factor * regional_factor)
                
                # Calculate revenue and profit
                revenue = quantity * product['price']
                profit = revenue * product['margin']
                
                sales_records.append({
                    'Date': current_date.strftime('%Y-%m-%d'),
                    'Month': current_date.strftime('%B'),
                    'Product': product['name'],
                    'Category': product['category'],
                    'Region': region,
                    'Quantity': quantity,
                    'Unit_Price': product['price'],
                    'Revenue': round(revenue, 2),
                    'Profit': round(profit, 2)
                })
    
    return pd.DataFrame(sales_records)

# Create dataset
df = generate_sales_data()

print("="*80)
print("BUSINESS SALES PERFORMANCE ANALYTICS")
print("="*80)
print(f"\nDataset Overview:")
print(f"Total Records: {len(df):,}")
print(f"Date Range: {df['Date'].min()} to {df['Date'].max()}")
print(f"\nFirst few records:")
print(df.head(10))

# Save dataset
df.to_csv('sales_data.csv', index=False)
print(f"\n✓ Dataset saved as 'sales_data.csv'")

# ============================================================================
# 1. KEY PERFORMANCE INDICATORS (KPIs)
# ============================================================================
print("\n" + "="*80)
print("1. KEY PERFORMANCE INDICATORS")
print("="*80)

total_revenue = df['Revenue'].sum()
total_profit = df['Profit'].sum()
total_orders = len(df)
avg_order_value = df['Revenue'].mean()
profit_margin = (total_profit / total_revenue) * 100

print(f"\n📊 Overall Business Metrics:")
print(f"   Total Revenue:        ${total_revenue:,.2f}")
print(f"   Total Profit:         ${total_profit:,.2f}")
print(f"   Profit Margin:        {profit_margin:.2f}%")
print(f"   Total Orders:         {total_orders:,}")
print(f"   Average Order Value:  ${avg_order_value:,.2f}")

# ============================================================================
# 2. REVENUE TRENDS ANALYSIS
# ============================================================================
print("\n" + "="*80)
print("2. REVENUE TRENDS ANALYSIS")
print("="*80)

# Monthly revenue and profit
monthly_performance = df.groupby('Month').agg({
    'Revenue': 'sum',
    'Profit': 'sum',
    'Quantity': 'sum'
}).round(2)

# Reorder by month
month_order = ['January', 'February', 'March', 'April', 'May', 'June',
               'July', 'August', 'September', 'October', 'November', 'December']
monthly_performance = monthly_performance.reindex(month_order)

print("\n📈 Monthly Performance:")
print(monthly_performance)

# Calculate growth rate
monthly_performance['Revenue_Growth'] = monthly_performance['Revenue'].pct_change() * 100
print("\n📊 Month-over-Month Growth Rates:")
print(monthly_performance[['Revenue', 'Revenue_Growth']].round(2))

# ============================================================================
# 3. TOP SELLING PRODUCTS
# ============================================================================
print("\n" + "="*80)
print("3. TOP SELLING PRODUCTS")
print("="*80)

# By Revenue
top_products_revenue = df.groupby('Product').agg({
    'Revenue': 'sum',
    'Quantity': 'sum',
    'Profit': 'sum'
}).sort_values('Revenue', ascending=False)

print("\n🏆 Top 10 Products by Revenue:")
print(top_products_revenue.head(10).round(2))

# By Quantity
top_products_quantity = df.groupby('Product')['Quantity'].sum().sort_values(ascending=False)
print("\n📦 Top 10 Products by Units Sold:")
print(top_products_quantity.head(10))

# ============================================================================
# 4. CATEGORY PERFORMANCE
# ============================================================================
print("\n" + "="*80)
print("4. CATEGORY PERFORMANCE ANALYSIS")
print("="*80)

category_performance = df.groupby('Category').agg({
    'Revenue': 'sum',
    'Profit': 'sum',
    'Quantity': 'sum'
}).sort_values('Revenue', ascending=False)

category_performance['Profit_Margin_%'] = (
    category_performance['Profit'] / category_performance['Revenue'] * 100
).round(2)

category_performance['Revenue_Share_%'] = (
    category_performance['Revenue'] / category_performance['Revenue'].sum() * 100
).round(2)

print("\n🏷️  Category Breakdown:")
print(category_performance.round(2))

# ============================================================================
# 5. REGIONAL PERFORMANCE
# ============================================================================
print("\n" + "="*80)
print("5. REGIONAL PERFORMANCE ANALYSIS")
print("="*80)

regional_performance = df.groupby('Region').agg({
    'Revenue': 'sum',
    'Profit': 'sum',
    'Quantity': 'sum'
}).sort_values('Revenue', ascending=False)

regional_performance['Revenue_Share_%'] = (
    regional_performance['Revenue'] / regional_performance['Revenue'].sum() * 100
).round(2)

print("\n🗺️  Regional Breakdown:")
print(regional_performance.round(2))

# Best performing region
best_region = regional_performance.index[0]
worst_region = regional_performance.index[-1]
print(f"\n✓ Best Performing Region:  {best_region} (${regional_performance.loc[best_region, 'Revenue']:,.2f})")
print(f"✗ Lowest Performing Region: {worst_region} (${regional_performance.loc[worst_region, 'Revenue']:,.2f})")

# ============================================================================
# 6. VISUALIZATIONS
# ============================================================================
print("\n" + "="*80)
print("6. GENERATING VISUALIZATIONS")
print("="*80)

# Create figure with subplots
fig = plt.figure(figsize=(16, 12))

# 1. Monthly Revenue Trend
ax1 = plt.subplot(3, 2, 1)
monthly_rev = df.groupby('Month')['Revenue'].sum().reindex(month_order)
ax1.plot(range(len(monthly_rev)), monthly_rev.values, marker='o', linewidth=2, markersize=8, color='#3b82f6')
ax1.fill_between(range(len(monthly_rev)), monthly_rev.values, alpha=0.3, color='#3b82f6')
ax1.set_xlabel('Month', fontsize=10, fontweight='bold')
ax1.set_ylabel('Revenue ($)', fontsize=10, fontweight='bold')
ax1.set_title('Monthly Revenue Trend', fontsize=12, fontweight='bold', pad=15)
ax1.grid(True, alpha=0.3)
ax1.set_xticks(range(len(month_order)))
ax1.set_xticklabels([m[:3] for m in month_order], rotation=45)

# 2. Top Products
ax2 = plt.subplot(3, 2, 2)
top_5 = top_products_revenue.head(5)
colors = ['#3b82f6', '#10b981', '#f59e0b', '#ef4444', '#8b5cf6']
bars = ax2.barh(range(len(top_5)), top_5['Revenue'].values, color=colors)
ax2.set_yticks(range(len(top_5)))
ax2.set_yticklabels(top_5.index)
ax2.set_xlabel('Revenue ($)', fontsize=10, fontweight='bold')
ax2.set_title('Top 5 Products by Revenue', fontsize=12, fontweight='bold', pad=15)
ax2.grid(True, alpha=0.3, axis='x')

# 3. Category Distribution
ax3 = plt.subplot(3, 2, 3)
cat_rev = category_performance['Revenue']
colors_pie = ['#3b82f6', '#10b981', '#f59e0b']
wedges, texts, autotexts = ax3.pie(cat_rev.values, labels=cat_rev.index, autopct='%1.1f%%',
                                     colors=colors_pie, startangle=90)
for autotext in autotexts:
    autotext.set_color('white')
    autotext.set_fontweight('bold')
ax3.set_title('Revenue Distribution by Category', fontsize=12, fontweight='bold', pad=15)

# 4. Regional Performance
ax4 = plt.subplot(3, 2, 4)
reg_data = regional_performance.sort_values('Revenue', ascending=True)
bars = ax4.barh(range(len(reg_data)), reg_data['Revenue'].values, color='#10b981')
ax4.set_yticks(range(len(reg_data)))
ax4.set_yticklabels(reg_data.index)
ax4.set_xlabel('Revenue ($)', fontsize=10, fontweight='bold')
ax4.set_title('Regional Performance', fontsize=12, fontweight='bold', pad=15)
ax4.grid(True, alpha=0.3, axis='x')

# 5. Monthly Profit Trend
ax5 = plt.subplot(3, 2, 5)
monthly_profit = df.groupby('Month')['Profit'].sum().reindex(month_order)
ax5.plot(range(len(monthly_profit)), monthly_profit.values, marker='s', linewidth=2, markersize=8, color='#10b981')
ax5.fill_between(range(len(monthly_profit)), monthly_profit.values, alpha=0.3, color='#10b981')
ax5.set_xlabel('Month', fontsize=10, fontweight='bold')
ax5.set_ylabel('Profit ($)', fontsize=10, fontweight='bold')
ax5.set_title('Monthly Profit Trend', fontsize=12, fontweight='bold', pad=15)
ax5.grid(True, alpha=0.3)
ax5.set_xticks(range(len(month_order)))
ax5.set_xticklabels([m[:3] for m in month_order], rotation=45)

# 6. Quantity Sold by Category
ax6 = plt.subplot(3, 2, 6)
cat_qty = category_performance['Quantity'].sort_values(ascending=True)
bars = ax6.barh(range(len(cat_qty)), cat_qty.values, color='#f59e0b')
ax6.set_yticks(range(len(cat_qty)))
ax6.set_yticklabels(cat_qty.index)
ax6.set_xlabel('Units Sold', fontsize=10, fontweight='bold')
ax6.set_title('Units Sold by Category', fontsize=12, fontweight='bold', pad=15)
ax6.grid(True, alpha=0.3, axis='x')

plt.tight_layout()
plt.savefig('sales_analysis_dashboard.png', dpi=300, bbox_inches='tight')
print("\n✓ Visualizations saved as 'sales_analysis_dashboard.png'")

# ============================================================================
# 7. KEY INSIGHTS & RECOMMENDATIONS
# ============================================================================
print("\n" + "="*80)
print("7. KEY INSIGHTS & ACTIONABLE RECOMMENDATIONS")
print("="*80)

print("\n📊 REVENUE INSIGHTS:")
print(f"   • Annual revenue reached ${total_revenue:,.2f} with a healthy {profit_margin:.1f}% profit margin")
print(f"   • Peak sales month: {monthly_performance['Revenue'].idxmax()} (${monthly_performance['Revenue'].max():,.2f})")
print(f"   • Lowest sales month: {monthly_performance['Revenue'].idxmin()} (${monthly_performance['Revenue'].min():,.2f})")

print("\n⭐ PRODUCT INSIGHTS:")
top_product = top_products_revenue.index[0]
top_category = category_performance.index[0]
print(f"   • Best seller: {top_product} (${top_products_revenue.loc[top_product, 'Revenue']:,.2f} revenue)")
print(f"   • Leading category: {top_category} ({category_performance.loc[top_category, 'Revenue_Share_%']:.1f}% of total revenue)")

print("\n🗺️  REGIONAL INSIGHTS:")
print(f"   • Strongest market: {best_region} region (${regional_performance.loc[best_region, 'Revenue']:,.2f})")
print(f"   • Growth opportunity: {worst_region} region needs targeted marketing")

print("\n💡 RECOMMENDATIONS:")
print("   1. Inventory Management:")
print("      → Increase stock for top 5 products to prevent stockouts")
print("      → Optimize inventory levels based on seasonal trends")
print("\n   2. Marketing Strategy:")
print(f"      → Launch targeted campaigns in {worst_region} region")
print("      → Promote high-margin products from Appliances category")
print("\n   3. Sales Optimization:")
print("      → Focus on Electronics category (highest revenue contributor)")
print("      → Implement cross-selling strategies for complementary products")
print("\n   4. Revenue Growth:")
print("      → Capitalize on mid-year sales peaks with special promotions")
print("      → Address revenue dips in low-performing months")

print("\n" + "="*80)
print("ANALYSIS COMPLETE")
print("="*80)
print("\nFiles Generated:")
print("  ✓ sales_data.csv - Raw sales dataset")
print("  ✓ sales_analysis_dashboard.png - Visual dashboard")
print("\nNext Steps:")
print("  → Upload to GitHub repository: FUTURE_DS_01")
print("  → Share insights on LinkedIn with visualizations")
print("  → Submit through Task Submission Portal")
print("="*80)