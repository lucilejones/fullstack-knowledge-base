


# notes from class 3/18/2024
Redis: https://redis.io/docs/get-started/
gem 'sidekiq'
gem 'sidekiq-scheduler'

resources for caching:
https://www.youtube.com/watch?v=jgpVdJB2sKQ&pp=ygUZcmVkaXMgdHV0b3JpYWwgb24gY2FjaGluZw%3D%3D
https://www.youtube.com/watch?v=-5RTyEim384&pp=ygUZcmVkaXMgdHV0b3JpYWwgb24gY2FjaGluZw%3D%3D
https://guides.rubyonrails.org/caching_with_rails.html
https://engineering.mrsool.co/using-redis-hashes-for-caching-in-ruby-on-rails-6dc53df293da

monthly_report_summary_job.rb:
class MonthlyReportSummaryJob

    def perform(*args)

        User.find_each do |user|
            total_likes = 0

            user.blogs.where(created_at: last_month.beginning_of_month..last_month.end_of_month).each do |blog|
                total_likes += blog.likes.count
            end

            MonthlySummary.create(user: user, month: last_month.beginning_of_month) do |summary|
                summary.total_likes = total_likes
            end

            puts "Mothly report created for #{user.username} with #{total_likes} likes."
        end
    end

end