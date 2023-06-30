#include <iostream>
#include <string>
#include <map>
#include <vector>
#include <utility>
#include <cmath>

struct Node {
    bool leaf;
    int l, r;
    int* n_r = nullptr;
    Node* parent;
    Node* suff_link;
    int num_of_leaf;
    std::map<char, Node*> childs{};

    Node(int l = -1, int r = -1, Node* par = nullptr, bool leaf = false);
    ~Node();

    int len();
    int end();
};

Node::Node(int l, int r, Node* par, bool leaf) {
    this->l = l;
    this->r = r;
    parent = par;
    n_r = nullptr;
    suff_link = nullptr;
    this->leaf = leaf;
    num_of_leaf = -1;
}

Node::~Node() {
    for (auto node : childs) {
        delete node.second;
    }
}

int Node::len() {
    return end() - l + 1;
}

int Node::end() {
    if (n_r == nullptr) {
        return r;
    }
    return *n_r;
}

class SuffixTree {
private:
    Node* root;
    Node* to_first_suff;
    Node* last_node;
    std::string string;
    int n_p;

    enum Link {
        DO_LINK,
        DO_LINK_TO_ROOT,
        DONT_LINK
    };

    void build();
    void do_phase(int &start_in_new_phase, int end, int &last_case_number, std::pair<Node*, int> &res_case3);
    std::pair<Node*, int> analysis_links(int start, int end);
    int analysis(std::pair<Node*, int> &pos, int end);
    std::pair<Node*, Link> case2(const std::pair<Node*, int> &pos, int start, int end, bool do_link);
    std::pair<Node*, int> case3(const std::pair<Node*, int> &pos, bool do_link);
    

public:
    SuffixTree(const std::string &str);
    ~SuffixTree();

    std::pair<Node*, int> find_path(int start, int end, Node* it);
    void depth_search(std::vector<int> &v, Node* it);
    int minimal_cut(int size);
};

SuffixTree::SuffixTree(const std::string &str) {
    root = new Node();
    to_first_suff = nullptr;
    last_node = nullptr;
    string = str;
    string += "{";
    n_p = 0;
    build();
}

SuffixTree::~SuffixTree() {
    delete root;
}

void SuffixTree::build() {
    to_first_suff = new Node(0, 0, root, true);
    to_first_suff->n_r = &n_p;
    to_first_suff->num_of_leaf = 0;
    last_node = to_first_suff;
    root->childs.insert(std::make_pair(string[0], to_first_suff));

    int start_in_new_phase = 1;
    int case_number = 1;
    std::pair<Node*, int> res_case3;
    for (int i = 1; i <= string.size() - 1; i++) {
        n_p++;
        do_phase(start_in_new_phase, i, case_number, res_case3);
    }
}

void SuffixTree::do_phase(int &start_in_new_phase, int end, int &case_number, std::pair<Node*, int> &res_case3) {
    bool do_link = false;
    for (int j = start_in_new_phase; j <= end; j++) {
        std::pair<Node*, int> pos;
        if (case_number == 3) {
            if (res_case3.second != 0) {
                pos = std::make_pair(res_case3.first, res_case3.second);
            } else {
                pos = find_path(end, end, res_case3.first);
            }
        } else {
            pos = analysis_links(j, end);
        }
        case_number = analysis(pos, end);

        if (case_number == 2) {
            std::pair<Node*, Link> res_of_case = case2(pos, j, end, do_link);
            last_node = res_of_case.first;
            if (res_of_case.second == DO_LINK) {
                do_link = true;
            } else if (res_of_case.second == DO_LINK_TO_ROOT) {
                last_node->suff_link = root;
                do_link = false;
            } else {
                do_link = false;
            }
            start_in_new_phase++;
        } else if (case_number == 3) {
            res_case3 = case3(pos, do_link);
            last_node = res_case3.first;
            break;
        }
    }
}

std::pair<Node*, int> SuffixTree::analysis_links(int start, int end) {
    if (last_node->parent == root || last_node->suff_link == root || last_node == root) {
        return find_path(start, end, root);
    } else {
        if (last_node->suff_link != nullptr) {
            return find_path(end, end, last_node->suff_link);
        } else {
            int d = last_node->len();
            return find_path(end - d, end, last_node->parent->suff_link);
        }
    }
}

std::pair<Node*, int> SuffixTree::find_path(int start, int end, Node* it) {
    if (it->childs.count(string[start]) == 1) {
        int d = it->childs[string[start]]->len() - (end - start);
        if (d >= 0) {
            return std::make_pair(it->childs[string[start]], end - start);
        } else {
            return find_path(start + it->childs[string[start]]->len(), end,
                             it->childs[string[start]]);
        }
    } else {
        return std::make_pair(it, -1 * (end - start + 1));
    }
    
}

int SuffixTree::analysis(std::pair<Node* , int> &pos, int end) {
    Node* node = pos.first;
    int p = pos.second;

    if (p < 0) {
        return 2;
    } else {
        if (node->len() == p && node->childs.size() > 1) {
            if (node->childs.count(string[end]) > 0) {
                pos.first = node->childs[string[end]];
                pos.second = 0;
                return 3;
            }
            return 2;
        } else if (node->len() > p) {
            if (string[node->l + p] == string[end]) {
                return 3;
            } else {
                return 2;
            }
        } else {
            return 3;
        }
    }
}

std::pair<Node*, SuffixTree::Link> SuffixTree::case2(const std::pair<Node*, int> &pos, int start, int end, bool do_link) {
    auto node = pos.first;
    auto p = pos.second;

    if (p >= 0) {
        if (p != node->len()) {
            Node* n = new Node(node->l, node->l + p - 1, node->parent, false);
            n->childs.insert(std::make_pair(string[node->l + p], node));
            node->parent->childs.erase(string[node->l]);
            node->parent->childs.insert(std::make_pair(string[n->l], n));
            node->l += p;
            node->parent = n;
            Node* nl = new Node(end, end, n, true);
            nl->n_r = &n_p;
            nl->num_of_leaf = start;
            n->childs.insert(std::make_pair(string[end], nl));

            if (do_link) {
                last_node->suff_link = n;
            }
            if (n->len() > 1) {
                return std::make_pair(n, DO_LINK);
            } else {
                if (n->parent != root) {
                    return std::make_pair(n, DO_LINK);
                }
                return std::make_pair(n, DO_LINK_TO_ROOT);
            }
        } else {
            Node* nl = new Node(end, end, node, true);
            nl->n_r = &n_p;
            nl->num_of_leaf = start;
            nl->parent->childs.insert(std::make_pair(string[end], nl));

            if (do_link) {
                last_node->suff_link = nl->parent;
            }
            return std::make_pair(nl->parent, DONT_LINK); 
        }
    } else {
        p *= -1;
        Node* n = new Node(end - p + 1, end, node, true);
        n->n_r = &n_p;
        n->num_of_leaf = start;
        n->parent->childs.insert(std::make_pair(string[end - p + 1], n));
        if (do_link) {
            last_node->suff_link = n->parent;
        }
        return std::make_pair(n->parent, DONT_LINK);
    }
}

std::pair<Node*, int> SuffixTree::case3(const std::pair<Node*, int> &pos, bool do_link) {
    if (do_link) {
        last_node->suff_link = pos.first->parent;
    }
    auto node = pos.first;
    auto p = pos.second;
    int case3_new_j;
    if (node->l + p == node->end()) {
        case3_new_j = 0;
    } else {
        case3_new_j = p + 1;
    }
    return std::make_pair(node, case3_new_j);
}

void SuffixTree::depth_search(std::vector<int> &v, Node* it) {
    if (it->leaf) {
        v.push_back(it->num_of_leaf);
        return;
    }
    for (auto child: it->childs) {
        depth_search(v, child.second);
    }
}

int SuffixTree::minimal_cut(int size) {
    Node* it = root;
    std::vector<int> all_indexes;
    for (int cur_sz = size; cur_sz >= 0;) {
        std::map<char, Node*>::iterator vertex_with_min_key = it->childs.begin();
        it = vertex_with_min_key->second;
        if (cur_sz <= it->len()) {
            depth_search(all_indexes, it);
            break;
        }
        cur_sz -= it->len();
    }
    int min = all_indexes[0];
    for (auto i: all_indexes) {
        if (i < min) {
            min = i;
        }
    }
    return min;
}

int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(nullptr);
    std::string str;
    std::cin >> str;
    if (str == "") {
        std::cout << "";
        return 0;
    } else {
        std::string new_str = str + str;
        SuffixTree s = SuffixTree(new_str);
        int start = s.minimal_cut(str.size());
        int end = start + str.size() - 1;
        for (int i = start; i <= end; i++) {
            std::cout << new_str[i];
        }
        std::cout << "\n";
        return 0;
    }
}
