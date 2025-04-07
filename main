#include <iostream>
#include <sstream>
#include <string>
#include <vector>
#include <map>
#include <regex>
#include <boost/asio.hpp>
#include <bsoncxx/json.hpp>
#include <bsoncxx/exception/exception.hpp>
#include <mongocxx/client.hpp>
#include <mongocxx/instance.hpp>
#include <mongocxx/uri.hpp>
#include <cmark.h>
#include <bsoncxx/stdx/string_view.hpp>
#include <bsoncxx/string/to_string.hpp>
#include <bsoncxx/types.hpp>
#include <bsoncxx/types/bson_value/view.hpp>

// ---------------------------------------------------------------------
// Forum categories or subforums, mapped to their IDs
// ---------------------------------------------------------------------
static std::map<std::string, int> forumCategories = {
    {"Software Vulnerabilities & Exploiting", 145},
    {"Malware", 65},
    {"Packers/Cryptors", 120},
    {"Cracking & Reversing", 100},
    {"Hardware Hacking & Phreaking", 5},
    {"Cryptography", 77},
    {"Messengers & Social Networks", 3},
    {"Anonymity & Security", 75},
    {"Spam, Traffic & Installs", 70},
    {"Social Engineering & Phishing", 84},
    {"AI & ML", 155},
    {"OSINT & Competitive Intelligence", 141},
    {"C/C++/C#/Rust/.NET/Java", 88},
    {"Delphi & Pascal", 87},
    {"Python", 144},
    {"PHP/JS/MySQL/Perl", 90},
    {"Other Languages", 40},
    {"System Administration", 8},
    {"Web Development", 61},
    {"News", 132},
    {"Bug Tracking", 122},
    {"Articles", 151},
    {"Manuals & Books", 153},
    {"Video Materials", 152},
    {"Databases", 140},
    {"Software", 126},
    {"Business & Investments", 125},
    {"Cryptocurrencies", 111},
    {"Gadgets & Hardware", 9},
    {"Media", 38},
    {"Behind The Scenes", 51},
    {"Chat", 29},
    {"Contact Administration", 39},
    {"Jabber Server", 143},
    {"File Sharing", 156},
    {"E-zine", 119}
};

// ---------------------------------------------------------------------
// A helper to parse subforum ID from a path like /forums/<id>/
// ---------------------------------------------------------------------
std::string parse_forum_id(const std::string& target) {
    std::regex forum_regex("^/forums/(\\d+)/?$");
    std::smatch match;
    if (std::regex_match(target, match, forum_regex) && match.size() > 1) {
        return match[1];
    }
    return "";
}

// ---------------------------------------------------------------------
// HTML rendering w/ cmark
// ---------------------------------------------------------------------
std::string render_markdown_to_html(const std::string& markdown_text) {
    cmark_node* doc = cmark_parse_document(markdown_text.c_str(), markdown_text.size(), CMARK_OPT_DEFAULT);
    char* html = cmark_render_html(doc, CMARK_OPT_UNSAFE);
    std::string result(html);
    free(html);
    cmark_node_free(doc);
    return result;
}

// ---------------------------------------------------------------------
// Fetch all forum threads from collection "xss.is-forums-<forum_id>" if using a mongodb backend
// ---------------------------------------------------------------------
std::vector<bsoncxx::document::view> get_forum_threads(mongocxx::client& client, const std::string& forum_id) {
    std::string collection_name = "xss.is-forums-" + forum_id;
    auto db = client["dbindex"];
    auto coll = db[collection_name];
    std::vector<bsoncxx::document::view> results;
    mongocxx::cursor cursor = coll.find({});
    for (auto&& doc : cursor) {
        results.push_back(doc);
    }
    return results;
}

// ---------------------------------------------------------------------
// HTML for the index page
// ---------------------------------------------------------------------
std::string build_index_page() {
    std::string html =
        "<!DOCTYPE html>"
        "<html>"
        "<head>"
        "<meta charset=\"UTF-8\">"
        "<meta name=\"viewport\" content=\"width=device-width, initial-scale=1\">"
        "<title>Forums Index</title>"
        "<style>"
        "body { margin: 0; padding: 20px; font-family: sans-serif; }"
        ".container { max-width: 800px; margin: 0 auto; }"
        "h1 { text-align: center; margin-bottom: 20px; }"
        ".category-list { list-style: none; padding: 0; }"
        ".category-item { margin: 8px 0; padding: 10px; border-radius: 4px; background: #f0f4f8; }"
        ".category-item a { text-decoration: none; color: #333; font-weight: bold; }"
        "@media (max-width: 600px) {"
        "  .category-item { font-size: 14px; }"
        "}"
        "</style>"
        "</head>"
        "<body>"
        "<div class=\"container\">"
        "<h1>Forum Categories</h1>"
        "<ul class=\"category-list\">";

    for (const auto& [name, id] : forumCategories) {
        html += "<li class=\"category-item\">"
                "<a href=\"/forums/" + std::to_string(id) + "\">" + name + "</a>"
                "</li>";
    }

    html += "</ul></div></body></html>";
    return html;
}

// ---------------------------------------------------------------------
// HTML for a specific subforum
// ---------------------------------------------------------------------
std::string build_forum_page(const std::vector<bsoncxx::document::view>& docs) {
    std::string html =
        "<!DOCTYPE html>"
        "<html>"
        "<head>"
        "<meta charset=\"UTF-8\">"
        "<meta name=\"viewport\" content=\"width=device-width, initial-scale=1\">"
        "<title>Forum Threads</title>"
        "<style>"
        "body { margin: 0; padding: 20px; font-family: sans-serif; }"
        ".container { max-width: 800px; margin: 0 auto; }"
        "h1 { text-align: center; }"
        ".thread { margin-bottom: 20px; padding: 10px; border: 1px solid #eee; border-radius: 4px; }"
        ".thread h2 { margin: 0 0 10px; font-size: 18px; }"
        ".thread-content { background: #f8f8f8; padding: 10px; border-radius: 4px; }"
        "</style>"
        "</head>"
        "<body>"
        "<div class=\"container\">"
        "<h1>Forum Threads</h1>";

    for (auto& doc : docs) {
        std::string title;
        std::string preview_content_markdown;


        if (doc["title"] && doc["title"].type() == bsoncxx::type::k_string) {
            title = bsoncxx::string::to_string(doc["title"].get_string().value);
        }

        if (doc["preview_content"] && doc["preview_content"].type() == bsoncxx::type::k_string) {
            preview_content_markdown = bsoncxx::string::to_string(doc["preview_content"].get_string().value);
        }

        std::string rendered = render_markdown_to_html(preview_content_markdown);

        html += "<div class=\"thread\">"
                "<h2>" + title + "</h2>"
                "<div class=\"thread-content\">" + rendered + "</div>"
                "</div>";
    }

    html += "</div></body></html>";
    return html;
}

int main() {
    static mongocxx::instance instance{};
    mongocxx::client client(mongocxx::uri("mongodb://localhost:27017"));

    boost::asio::io_context io_context;
    boost::asio::ip::tcp::acceptor acceptor(io_context, boost::asio::ip::tcp::endpoint(boost::asio::ip::tcp::v4(), 9999));

    std::cout << "Server running on http://localhost:9999 ...\n";

    for (;;) {
        boost::asio::ip::tcp::socket socket(io_context);
        acceptor.accept(socket);

        boost::asio::streambuf request_buf;
        boost::system::error_code ec;
        size_t bytes_transferred = boost::asio::read_until(socket, request_buf, "\r\n\r\n", ec);

        if (ec && ec != boost::asio::error::not_found) {
            socket.close();
            continue;
        }

        std::istream request_stream(&request_buf);
        std::string request_line;
        std::getline(request_stream, request_line);

        std::string method, target, version;
        {
            std::istringstream iss(request_line);
            iss >> method >> target >> version;
        }

        std::string response;
        if (method == "GET") {
            if (target == "/") {
                std::string page = build_index_page();
                response = "HTTP/1.1 200 OK\r\n"
                           "Content-Type: text/html; charset=UTF-8\r\n"
                           "Content-Length: " + std::to_string(page.size()) + "\r\n"
                           "\r\n" + page;
            } else {
                std::string forum_id = parse_forum_id(target);
                if (!forum_id.empty()) {
                    std::vector<bsoncxx::document::view> docs = get_forum_threads(client, forum_id);
                    std::string page = build_forum_page(docs);
                    response = "HTTP/1.1 200 OK\r\n"
                               "Content-Type: text/html; charset=UTF-8\r\n"
                               "Content-Length: " + std::to_string(page.size()) + "\r\n"
                               "\r\n" + page;
                } else {
                    std::string not_found = "<html><body><h1>Not Found</h1></body></html>";
                    response = "HTTP/1.1 404 Not Found\r\n"
                               "Content-Type: text/html; charset=UTF-8\r\n"
                               "Content-Length: " + std::to_string(not_found.size()) + "\r\n"
                               "\r\n" + not_found;
                }
            }
        } else {
            std::string not_allowed = "<html><body><h1>Method Not Allowed</h1></body></html>";
            response = "HTTP/1.1 405 Method Not Allowed\r\n"
                       "Content-Type: text/html\r\n"
                       "Content-Length: " + std::to_string(not_allowed.size()) + "\r\n"
                       "\r\n" + not_allowed;
        }

        boost::asio::write(socket, boost::asio::buffer(response), ec);
        socket.close();
    }
    return 0;
}
