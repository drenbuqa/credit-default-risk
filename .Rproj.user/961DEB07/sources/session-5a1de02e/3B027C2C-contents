# Libraries
library(tidyverse)
library(ggplot2)
library(corrplot)
library(skimr)
library(readxl)

# Load data
df <- read_excel("data/credit_default.xls", sheet = 1, skip = 1)

# Quick overview
skim(df)

# Clean column names
df <- df %>%
  rename(default = `default payment next month`)

# Check default rate
default_rate <- mean(df$default)
cat("Default rate:", round(default_rate * 100, 1), "%\n") # Default rate: 22.1 %

# Convert categorical variables to factors
df <- df %>%
  mutate(
    SEX = factor(SEX, labels = c("Male", "Female")),
    EDUCATION = factor(EDUCATION),
    MARRIAGE = factor(MARRIAGE),
    default = factor(default, labels = c("No Default", "Default"))
  )

# Create outputs folder if it doesn't exist
dir.create("outputs", showWarnings = FALSE)

# Plot 1 - Default rate overview
ggplot(df, aes(x = default, fill = default)) +
  geom_bar() +
  geom_text(stat = "count", aes(label = after_stat(count)), vjust = -0.5) +
  scale_fill_manual(values = c("No Default" = "#2196F3", "Default" = "#F44336")) +
  labs(title = "Credit Default Distribution",
       x = "Default Status", y = "Number of Clients") +
  theme_minimal() +
  theme(legend.position = "none")

ggsave("outputs/01_default_distribution.png", width = 8, height = 5)

# Plot 2 - Default rate by education
ggplot(df, aes(x = EDUCATION, fill = default)) +
  geom_bar(position = "fill") +
  scale_fill_manual(values = c("No Default" = "#2196F3", "Default" = "#F44336")) +
  scale_y_continuous(labels = scales::percent) +
  labs(title = "Default Rate by Education Level",
       x = "Education (1=Graduate, 2=University, 3=High School)",
       y = "Proportion", fill = "Status") +
  theme_minimal()

ggsave("outputs/02_default_by_education.png", width = 8, height = 5)

# Plot 3 - Default rate by age group
df %>%
  mutate(age_group = cut(AGE, breaks = c(20, 30, 40, 50, 80),
                         labels = c("21-30", "31-40", "41-50", "51+"))) %>%
  group_by(age_group, default) %>%
  summarise(count = n(), .groups = "drop") %>%
  group_by(age_group) %>%
  mutate(pct = count / sum(count)) %>%
  filter(default == "Default") %>%
  ggplot(aes(x = age_group, y = pct, fill = age_group)) +
  geom_col() +
  scale_y_continuous(labels = scales::percent) +
  scale_fill_brewer(palette = "Blues") +
  labs(title = "Default Rate by Age Group",
       x = "Age Group", y = "Default Rate") +
  theme_minimal() +
  theme(legend.position = "none")

ggsave("outputs/03_default_by_age.png", width = 8, height = 5)

# Plot 4 - Credit limit distribution by default status
ggplot(df, aes(x = LIMIT_BAL, fill = default)) +
  geom_histogram(bins = 50, alpha = 0.7, position = "identity") +
  scale_fill_manual(values = c("No Default" = "#2196F3", "Default" = "#F44336")) +
  scale_x_continuous(labels = scales::comma) +
  labs(title = "Credit Limit Distribution by Default Status",
       x = "Credit Limit (NT$)", y = "Count", fill = "Status") +
  theme_minimal()

ggsave("outputs/04_credit_limit_distribution.png", width = 8, height = 5)

# Plot 5 - Correlation heatmap of numeric variables
numeric_df <- df %>%
  select(LIMIT_BAL, AGE, PAY_0, PAY_2, PAY_3, PAY_4, PAY_5, PAY_6,
         BILL_AMT1, PAY_AMT1, default) %>%
  mutate(default = as.numeric(default) - 1)

cor_matrix <- cor(numeric_df)

png("outputs/05_correlation_heatmap.png", width = 900, height = 800)
corrplot(cor_matrix,
         method = "color",
         type = "upper",
         tl.cex = 0.8,
         tl.col = "black",
         addCoef.col = "black",
         number.cex = 0.6,
         col = colorRampPalette(c("#F44336", "white", "#2196F3"))(200),
         title = "Correlation Matrix",
         mar = c(0, 0, 2, 0))
dev.off()

# Plot 6 - Payment delay vs default rate
df %>%
  group_by(PAY_0, default) %>%
  summarise(count = n(), .groups = "drop") %>%
  group_by(PAY_0) %>%
  mutate(pct = count / sum(count)) %>%
  filter(default == "Default") %>%
  ggplot(aes(x = factor(PAY_0), y = pct, fill = pct)) +
  geom_col() +
  scale_y_continuous(labels = scales::percent) +
  scale_fill_gradient(low = "#2196F3", high = "#F44336") +
  labs(title = "Default Rate by Most Recent Payment Status (PAY_0)",
       subtitle = "-2/-1 = paid on time, 1+ = months delayed",
       x = "Payment Status", y = "Default Rate") +
  theme_minimal() +
  theme(legend.position = "none")

ggsave("outputs/06_default_by_payment_status.png", width = 8, height = 5)

# Export clean data for Python
write.csv(df %>% mutate(default = as.numeric(default) - 1),
          "outputs/credit_default_clean.csv", row.names = FALSE)

cat("R analysis complete. Clean data exported to outputs/credit_default_clean.csv\n")